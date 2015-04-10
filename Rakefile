#!/usr/bin/env ruby

require 'open-uri'
require 'csv'
require 'nokogiri'

Dir.glob('../treemap/lib/treemap/*.rake').each { |r| import r }

desc "Generate the sql file"
task :build => [:has_required_files] do
  Dir.chdir "zip_files"

  # Unzip files
  `ls *.zip | parallel unzip -a -o`

  # Create SQL scripts
  Dir["*.shp"].each do |shp|
    file_head = shp.gsub(/.shp/, '')
    `shp2pgsql -s 4326 -c -W UTF-8 $shp #{file_head} > #{file_head}.sql`
  end

  create_postgis_db "tree_dataset"

  # Import tree_data into sql
  Dir["*.sql"].each do |shp|
    puts `sudo -u postgres psql tree_dataset < #{shp}`
  end

  sql_cmd(
    "tree_dataset",
    "create table trees( id integer, latin_name varchar(100), common_name varchar(100), area double precision, geom geometry(MultiPolygon, 4326))")

  sql_cmd(
    "tree_dataset",
    "CREATE INDEX CONCURRENTLY tree_geom_idx ON trees USING gist(geom)")

  CSV.foreach("../tree_files.csv") do |row|
    table = row[0].gsub(/.zip/, '')

    sql_cmd(
      "tree_dataset",
      "insert into trees (latin_name, common_name, area, geom) select '#{row[1].strip}', '#{row[2].strip}', area, geom from #{table}")

    sql_cmd(
      "tree_dataset",
      "drop table #{table}")
  end
end

desc "Clean up everything"
task :clean do
  Dir.chdir "zip_files"

  `ls | grep -v .zip | xargs rm`
  drop_db "tree_dataset"
end

desc "Dump the file"
task :dump do
  file = "trees-#{Time.now.strftime("%Y-%m-%d")}.sql"

  before_dump = `sudo -u postgres psql tree_dataset -c "select count(*) from trees"`
  `sudo -u postgres pg_dump --no-acl --no-owner tree_dataset > #{file}`
  create_postgis_db "tree_dataset_test"
  `sudo -u postgres psql tree_dataset_test < #{file}`
  after_dump = `sudo -u postgres psql tree_dataset_test -c "select count(*) from trees"`
  drop_db "tree_dataset_test"

  if before_dump != after_dump
    puts "Invalid"
    puts "Before:"
    puts before_dump
    puts "After:"
    puts after_dump
  end
  # TODO: Test import of the database
end

desc "Dump tree_files.csv"
task :dump_tree_files do
  doc = Nokogiri::HTML(open("info.html"))
  doc.xpath("//tr").to_a[1..-1].each do |tr|
    latin_name = tr.children[2].text
    common_name = tr.children[3].text
    file = tr.children[1].children[0].attributes["href"].value

    puts "#{file}, #{latin_name}, #{common_name}"
  end
end
