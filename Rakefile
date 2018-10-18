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

desc "Dump the RDFS/OWL ontology in Turtle format"
task 'dump:ttl' do
  require 'rdf'
  require 'rdf/turtle'
  puts rdf.dump(:turtle, base_uri: BASE_URI, prefixes: rdf_prefixes)
end

desc "Dump the RDFS/OWL ontology in N-Triples format"
task 'dump:nt' do
  require 'rdf'
  require 'rdf/ntriples'
  puts rdf.dump(:ntriples)
end

desc "Dump the RDFS/OWL ontology"
task :dump => 'dump:ttl'
