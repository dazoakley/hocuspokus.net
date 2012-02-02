---
layout: post
title: ! 'The Black Art of BioMart: Ontology Searching'
permalink: /2011/04/the-black-art-of-biomart-ontology-searching
tags:
- biomart
- ontology
- search
- ruby
---

We use [BioMart](http://www.biomart.org) a **lot** at my workplace, and one of the recent things I was tasked with
doing was adding full ontology searching to one of our datasets - to be specific, MP ontology searching over some
MP annotated phenotyping data.  The desired functionality is so that a user can come in with an MP ID and retrieve
any phenotyping data that has been annotated to that ID, or one of it's children in the MP tree.

Time for a small disclaimer... ;-)

I'm not the first person to do this kind of stuff, it's been about for a long time (GO searching has been in the
Ensembl mart as long as I can remember), but i've never seen anyone write down how you do it.  Also, please note
that this info is for **BioMart version 0.7** - the configuration mechanisms in the new 0.8 series are quite
different (and i've not had a proper chance to play with 0.8 yet).

I've tried to document as much as I can but obviously when it comes to the XML examples
you'll have to take my code as a guide and apply the same principles to your datasets in MartEditor.  Here we go...

The process boils down into the following tasks:

* Set-up and configure an ontology dataset
  * Create the (mp\_ontology) staging table
  * Populate the staging table with the required data
  * Configure the (mp\_ontology) dataset
* Configure to our target dataset to use the ontology dataset

## The Ontology Dataset

In order to enable full ontology searching, you must have a separate ontology dataset so that your target dataset
(in my case the phenotyping data) can point into it.  It's a relatively simple single table BioMart dataset...

### The Ontology Staging Table

The staging table for the ontology dataset takes the following form:

{% highlight text %}
mysql> describe mp_ontology;
+-------------+--------------+------+-----+---------+-------+
| Field       | Type         | Null | Key | Default | Extra |
+-------------+--------------+------+-----+---------+-------+
| parent_id   | varchar(50)  | NO   | PRI | NULL    |       |
| parent_term | varchar(255) | NO   |     | NULL    |       |
| child_id    | varchar(50)  | NO   | PRI | NULL    |       |
| child_term  | varchar(255) | NO   |     | NULL    |       |
+-------------+--------------+------+-----+---------+-------+
{% endhighlight %}

Basically you fill it with mapping data for each parent ontology term.  The data for each parent ID should include
a mapping to itself, and **all** children below it e.g. for the root MP:00000001 there should be row where the child ID
is also MP:00000001 and then rows for every single other MP term.  It's all a bit redundant, but this is what allows
the mart to do the full ontology tree filtering.

Here's a little script I knocked together to create and populate the mp_ontology table, it's written in ruby, and uses
the [ols](https://rubygems.org/gems/ols) gem to get the ontology terms and ID's (**NOTE:** you'll need a local copy
of the [OLS](http://www.ebi.ac.uk/ontology-lookup/) database in order to use this script - more details are in the
[gem documentation](http://rubydoc.info/gems/ols/0.0.1/frames)).

{% highlight ruby %}
#!/usr/bin/env ruby

require 'logger'
require 'optparse'
require 'ols'
require 'sequel'
require 'mysql2'

# Set the script options

@log        = Logger.new(STDOUT)
@log.level  = Logger::INFO
@debug      = false
@table_name = nil
@root_term  = nil

OptionParser.new do |opts|
  opts.banner = "Usage: build_ontology_table.rb [options]"
  opts.on("-d", "--debug", "Show debug output")                    { |d| @debug = d }
  opts.on("-t", "--table TABLE_NAME", "The table to create")       { |table| @table_name = table.to_sym }
  opts.on("-r", "--root_term ROOT_TERM", "The ontology root term") { |term| @root_term = term }
  opts.on_tail("-h", "--help", "Display help")                     { puts opts; exit; }
end.parse!

raise "You must define a --table to insert into!" if @table_name.nil?
raise "You must define a --root_term!"            if @root_term.nil?

# Connect to the BioMart database

@log.info("Connecting to BioMart staging database")
BIOMART_DB = Sequel.connect(
  :adapter  => 'mysql2',
  :host     => '127.0.0.1',
  :port     => 3306,
  :database => 'mart_staging',
  :user     => 'mart',
  :password => 'mart',
  :test     => true
)
BIOMART_DB.loggers << @log if @debug

# Now get on with stuff...

@log.info("Creating table '#{@table_name}' in staging database")

begin
  BIOMART_DB.drop_table(@table_name)
rescue
end

BIOMART_DB.create_table @table_name do
  String :parent_id, :size => 50, :null => false
  String :parent_term, :null => false
  String :child_id, :size => 50, :null => false
  String :child_term, :null => false
  primary_key [:parent_id,:child_id]
end

def insert_self( ont )
  ds = BIOMART_DB[@table_name].filter( :parent_id => ont.term, :child_id => ont.term )
  if ds.count == 0
    BIOMART_DB[@table_name].insert(
      :parent_id   => ont.term,
      :parent_term => ont.term_name,
      :child_id    => ont.term,
      :child_term  => ont.term_name
    )
  end
end

def insert_children( ont )
  terms      = ont.all_child_terms
  term_names = ont.all_child_names

  terms.each_index do |index|
    ds = BIOMART_DB[@table_name].filter( :parent_id => ont.term, :child_id => terms[index] )
    if ds.count == 0
      BIOMART_DB[@table_name].insert(
        :parent_id   => ont.term,
        :parent_term => ont.term_name,
        :child_id    => terms[index],
        :child_term  => term_names[index]
      )
    end
  end

  ont.children.each do |child|
    insert_self( child )
    insert_children( child ) if child.has_children?
  end
end

@log.info("Getting root term and building ontology tree")
tree = OLS::OntologyTerm.new(@root_term)
tree.build_tree

@log.info("Loading terms to staging table")
insert_self( tree )
insert_children( tree )

@log.info("Done. :)")
{% endhighlight %}

You run the script with the following options (just be warned, it takes a while for a large ontology):

{% highlight text %}
ruby build_ontology_table.rb --table mp_ontology --root_term MP:0000001 --debug
{% endhighlight %}


### Configuring the Ontology Dataset

Now that we have our ontology staging table, run through the normal MartBuilder process to setup and
generate the SQL for creating your new mp\_ontology dataset.  (If you want to make the searches a touch more
efficient, add indexes on the parent\_id and parent\_term fields whilst in MartBuilder).  Before moving on
run the build SQL to generate your dataset tables.  Now, onto MartEditor.

In MartEditor we need to do the following:

* Set-up filters for the parent\_id and parent\_term fields
* Set-up an attribute for the child\_id field
* Set-up an exportable on the child\_id attribute

Here's my MartEditor XML (i'll let you change the specific details for your means or click this through in
MartEditor):

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE DatasetConfig>
<DatasetConfig dataset="mp_ontology" datasetID="13" displayName="Mammalian Phenotype (MP) Ontology" hideDisplay="true" interfaces="default" internalName="default" martUsers="default" modified="2011-04-19 10:06:02" softwareVersion="0.6" template="mp_ontology" type="TableSet" visible="0">
  <MainTable>mp_ontology__mp_ontology__main</MainTable>
  <Key>child_id_2010_key</Key>
  <Exportable attributes="child_mp_id" internalName="mp_id" linkName="mp_id" name="mp_id" type="link"/>
  <FilterPage displayName="FILTERS" internalName="filters">
    <FilterGroup displayName="FILTERS" internalName="filters">
      <FilterCollection displayName="mp ontology" internalName="mp_ontology">
        <FilterDescription displayName="MP ID" displayType="text" field="parent_id_2010_key" internalName="mp_id_display" key="child_id_2010_key" legal_qualifiers="=" qualifier="=" tableConstraint="main" type="text"/>
        <FilterDescription displayName="MP Term" displayType="text" field="parent_term_2010" internalName="mp_term" key="child_id_2010_key" legal_qualifiers="=" qualifier="=" tableConstraint="main" type="text"/>
      </FilterCollection>
    </FilterGroup>
  </FilterPage>
  <AttributePage displayName="ATTRIBUTES" internalName="attributes" outFormats="html,txt,csv,tsv,xls">
    <AttributeGroup displayName="FEATURES" internalName="features">
      <AttributeCollection displayName="mp ontology" internalName="mp_ontology">
        <AttributeDescription displayName="Parent ID" field="parent_id_2010_key" internalName="parent_mp_id" key="child_id_2010_key" maxLength="255" tableConstraint="main"/>
        <AttributeDescription displayName="Parent Term" field="parent_term_2010" internalName="parent_mp_term" key="child_id_2010_key" maxLength="255" tableConstraint="main"/>
        <AttributeDescription displayName="Child ID" field="child_id_2010_key" internalName="child_mp_id" key="child_id_2010_key" maxLength="255" tableConstraint="main"/>
        <AttributeDescription displayName="Child Term" field="child_term_2010" internalName="child_mp_term" key="child_id_2010_key" maxLength="255" tableConstraint="main"/>
      </AttributeCollection>
    </AttributeGroup>
  </AttributePage>
</DatasetConfig>
{% endhighlight %}

The **main** gist of it is this:

* The filters are called:
  * mp\_id\_display - this filters on the parent MP ID (**NOTE:** the '\_display' in the name is important - it's to stop a name clash later on)
  * mp\_term - this filters on the parent MP Term
* The important attribute is:
  * child\_mp\_id - this returns on the Child MP ID's
* The exportable:
  * is called mp\_id
  * points at the child\_mp\_id attribute

That's the mp\_ontology dataset configured we now need to configure our target dataset...

## Configuring Our Target Dataset

The final step now is to configure the target dataset to use the ontology dataset for its ontology searches.
So, what needs to be done is the following:

* Add a hidden filter on the mp\_id field that is already present in your dataset
* Add an importable on tthis mp\_id field - this will link it to the exportable in the ontology dataset
* Add **pointer** filters to the mp\_id\_display and mp\_term filters from the mp\_ontology dataset

First up, the hidden filter...  when I setup my example I created this in it's own 'FilterPage' to separate from
everything else in my config, (just in case I screwed things up) here's the XML:

{% highlight xml %}
<FilterPage hideDisplay="true" internalName="mp_ontology">
  <FilterGroup hideDisplay="true" internalName="mp_ontology">
    <FilterCollection hideDisplay="true" internalName="param_level_heatmap_mp_id">
      <FilterDescription displayName="MP ID" displayType="text" field="mp_term" hideDisplay="true" internalName="param_level_heatmap_mp_id" key="id_1024_key" legal_qualifiers="=" qualifier="=" tableConstraint="phenotyping__param_level_heatmap__dm" type="text"/>
    </FilterCollection>
  </FilterGroup>
</FilterPage>
{% endhighlight %}

Then the importable:

{% highlight xml %}
<Importable filters="param_level_heatmap_mp_id" internalName="mp_id" linkName="mp_id" name="mp_id" type="link"/>
{% endhighlight %}

And finally the two pointer filters:

{% highlight xml %}
<FilterCollection displayName="MP ID" internalName="param_level_heatmap_mp_id_display">
  <FilterDescription displayName="MP ID" internalName="param_level_heatmap_mp_id_display" pointerDataset="mp_ontology" pointerFilter="mp_id_display" pointerInterface="default" tableConstraint="phenotyping__"/>
</FilterCollection>
<FilterCollection displayName="MP Term" internalName="param_level_heatmap_mp_term">
  <FilterDescription displayName="MP Term" internalName="param_level_heatmap_mp_term" pointerDataset="mp_ontology" pointerFilter="mp_term" pointerInterface="default" tableConstraint="phenotyping__"/>
</FilterCollection>
{% endhighlight %}

And that's it - we now have MP searching! :)
