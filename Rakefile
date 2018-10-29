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

def rdf
  require 'rdf'
  require 'rdf/ntriples'
  require 'rdf/reasoner'
  require 'rdf/turtle'
  require 'rdf/xsd'
  RDF::Reasoner.apply(:rdfs, :owl)
  RDF::Graph.new do |graph|
    graph.load('rdf/classes.ttl')
    graph.load('rdf/properties.ttl')
    graph.entail!
  end
end

def rdf_classes
  require 'rdf'
  sampo = RDF::Vocabulary.new(BASE_URI)
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
  sampo = RDF::Vocabulary.new(BASE_URI)
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
