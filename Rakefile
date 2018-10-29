# This is free and unencumbered software released into the public domain.

BASE_URI = 'https://rdf.sampoapp.org/'

def rdf_prefixes
  require 'rdf/vocab'
  {
    doap: RDF::Vocab::DOAP.to_uri,
    foaf: RDF::Vocab::FOAF.to_uri,
    owl:  RDF::OWL.to_uri,
    rdf:  RDF.to_uri,
    rdfs: RDF::RDFS.to_uri,
  }
end

def sampo
  RDF::Vocabulary.new(BASE_URI)
end

def rdf
  require 'rdf'
  require 'rdf/ntriples'
  #require 'rdf/reasoner'
  require 'rdf/turtle'
  require 'rdf/xsd'
  #RDF::Reasoner.apply(:rdfs, :owl)
  RDF::Graph.new do |graph|
    graph.load('rdf/classes.ttl')
    graph.load('rdf/properties.ttl')
    #graph.entail!
  end
end

def rdf_classes
  require 'rdf'
  query = RDF::Query.new({
    :class => {
      RDF::RDFS.isDefinedBy => RDF::URI(BASE_URI),
      RDF::RDFS.subClassOf => sampo.Entity,
    },
  })
  result = query.execute(rdf).map { |row| row[:class].relativize(BASE_URI).to_s }
  result.sort.map(&:to_sym)
end

def rdf_subclasses
  require 'rdf'
  query = RDF::Query.new({
    :class => {
      RDF::RDFS.isDefinedBy => RDF::URI(BASE_URI),
      RDF::RDFS.subClassOf => :superclass,
    },
    :superclass => {
      RDF::RDFS.subClassOf => sampo.Entity,
    },
  })
  result = query.execute(rdf).map { |row| row[:class].relativize(BASE_URI).to_s }
  result.sort.map(&:to_sym)
end

def rdf_properties
  require 'rdf'
  query = RDF::Query.new({
    :property => {
      RDF::RDFS.isDefinedBy => RDF::URI(BASE_URI),
      RDF.type => RDF.Property,
    },
  })
  result = query.execute(rdf).map { |row| row[:property].relativize(BASE_URI).to_s }
  result.sort.map(&:to_sym)
end

desc "Dump the RDFS/OWL ontology in Turtle format"
task 'dump:ttl' do
  puts rdf.dump(:turtle, base_uri: BASE_URI, prefixes: rdf_prefixes)
end

desc "Dump the RDFS/OWL ontology in N-Triples format"
task 'dump:nt' do
  puts rdf.dump(:ntriples)
end

desc "Dump the RDFS/OWL ontology"
task :dump => 'dump:ttl'

desc "List all namespace prefixes in the ontology."
task 'list:prefixes' do
  rdf_prefixes.each do |k, v|
    puts "#{k}\t#{v}"
  end
end

desc "List all superclasses in the ontology."
task 'list:classes' do
  rdf_classes.each do |row|
    puts row
  end
end

desc "List all subclasses in the ontology."
task 'list:subclasses' do
  rdf_subclasses.each do |row|
    puts row
  end
end

desc "List all properties in the ontology."
task 'list:props' do
  rdf_properties.sort.each do |row|
    puts row
  end
end

desc "Generate the Dart code for EntityClasses.top."
task 'gen:classes:top' do
  require 'rdf'
  RDF::Query.execute(rdf, {
    :class => {
      RDF::RDFS.isDefinedBy => RDF::URI(BASE_URI),
      RDF::RDFS.subClassOf => sampo.Entity,
      sampo.id => :id,
      sampo.icon => :icon,
    },
  })
  .reject { |row| row.icon.to_s.empty? } # skip classes w/o icons
  .sort { |row1, row2| row1.id <=> row2.id }.each do |row|
    puts %Q<"#{row.id}",>
  end
end

desc "Generate the Dart code for EntityClasses.all."
task 'gen:classes:all' do
  require 'rdf'
  RDF::Query.execute(rdf, {
    :class => {
      RDF::RDFS.isDefinedBy => RDF::URI(BASE_URI),
      RDF::RDFS.subClassOf => sampo.Entity,
      sampo.id => :id,
      sampo.icon => :icon,
    },
  })
  .reject { |row| row.icon.to_s.empty? } # skip classes w/o icons
  .sort { |row1, row2| row1.id <=> row2.id }.each do |row|
    subclasses = RDF::Query.execute(rdf, {
      :class => {
        RDF::RDFS.isDefinedBy => RDF::URI(BASE_URI),
        RDF::RDFS.subClassOf => row[:class],
        sampo.id => :id,
      },
    }).map { |row| row.id.to_s }.sort

    puts %Q<"#{row.id}": EntityClass("#{row.id}",>
    puts %Q<  icon: Icons.#{row.icon},>
    puts %Q<  superclass: null,>
    if subclasses.empty?
      puts %Q<  subclasses: <String>[],>
    else
      puts %Q<  subclasses: <String>[>
      subclasses.each do |subclass|
        puts %Q<    "#{subclass}",>
      end
      puts %Q<  ],>
    end
    puts %Q<),>
  end
end

desc "Generate the SQLite schema."
task 'gen:sqlite' do
  require 'rdf'
  RDF::Query.execute(rdf, {
    :class => {
      RDF::RDFS.isDefinedBy => RDF::URI(BASE_URI),
      RDF::RDFS.subClassOf => sampo.Entity,
      sampo.id => :id,
      sampo.icon => :icon,
    },
  })
  .reject { |row| row.icon.to_s.empty? } # skip classes w/o icons
  .sort { |row1, row2| row1.id <=> row2.id }.each do |row|
    subclasses = RDF::Query.execute(rdf, {
      :class => {
        RDF::RDFS.isDefinedBy => RDF::URI(BASE_URI),
        RDF::RDFS.subClassOf => row[:class],
        sampo.id => :id,
      },
    }).map { |row| row.id.to_s }.sort

    top_table = "data_#{row.id}"
    puts
    puts "-"*80
    puts
    puts %Q<DROP TABLE IF EXISTS #{top_table};>
    puts %Q<CREATE TABLE #{top_table} (>
    puts %Q<  id INTEGER PRIMARY KEY NOT NULL>
    # TODO
    puts %Q<);>

    unless subclasses.empty?
      subclasses.each do |subclass|
        sub_table = "#{top_table}_#{subclass}"
        puts
        puts %Q<DROP TABLE IF EXISTS #{sub_table};>
        puts %Q<CREATE TABLE #{sub_table} (id INTEGER PRIMARY KEY NOT NULL);>
      end
    end
  end
end
