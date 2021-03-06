#
# Plist Rakefile
#
# Based heavily on Geoffrey Grosenbach's Rakefile for gruff.
# Includes whitespace-fixing task based on code from Typo.
#
# Copyright 2006-2010 Ben Bleything and Patrick May
# Copyright 2015-2016 Albert Wang and Cellabus, Inc.
# Distributed under the MIT License
#

require 'fileutils'
require 'rubygems'
require 'rake'
require 'rake/testtask'
require 'rake/packagetask'
require 'rake/contrib/rubyforgepublisher'
require 'rubygems/package_task'

$:.unshift(File.dirname(__FILE__) + "/lib")
require 'plist'

PKG_NAME      = 'plist'
PKG_VERSION   = Plist::VERSION
PKG_FILE_NAME = "#{PKG_NAME}-#{PKG_VERSION}"

RELEASE_NAME  = "REL #{PKG_VERSION}"

RUBYFORGE_PROJECT = "plist"
RUBYFORGE_USER    = ENV['RUBYFORGE_USER']

TEST_FILES    = Dir.glob('test/test_*')
TEST_ASSETS   = Dir.glob('test/assets/*')
LIB_FILES     = Dir.glob('lib/**/*')
RELEASE_FILES = [ "Rakefile", "README.rdoc", "CHANGELOG", "LICENSE" ] + LIB_FILES + TEST_FILES + TEST_ASSETS

task :default => [ :test ]
# Run the unit tests
Rake::TestTask.new do |t|
  t.libs << "test"
  t.test_files = ['test/test']
  t.verbose = true
end

desc "Clean pkg and rdoc; remove .bak files"
task :clean => [ :clobber_rdoc, :clobber_package ] do
  puts cmd = "find . -type f -name *.bak -delete"
  `#{cmd}`
end

desc "Strip trailing whitespace and fix newlines for all release files"
task :fix_whitespace => [ :clean ] do
  RELEASE_FILES.reject {|i| i =~ /assets/}.each do |filename|
    next if File.directory? filename

    File.open(filename) do |file|
      newfile = ''
      needs_love = false

      file.readlines.each_with_index do |line, lineno|
        if line =~ /[ \t]+$/
          needs_love = true
          puts "#{filename}: trailing whitespace on line #{lineno}"
          line.gsub!(/[ \t]*$/, '')
        end

        if line.chomp == line
          needs_love = true
          puts "#{filename}: no newline on line #{lineno}"
          line << "\n"
        end

        newfile << line
      end

      if needs_love
        tempname = "#{filename}.new"

        File.open(tempname, 'w').write(newfile)
        File.chmod(File.stat(filename).mode, tempname)

        FileUtils.ln filename, "#{filename}.bak"
        FileUtils.ln tempname, filename, :force => true
        File.unlink(tempname)
      end
    end
  end
end

desc "Copy documentation to rubyforge"
task :update_rdoc => [ :rdoc ] do
  Rake::SshDirPublisher.new("#{RUBYFORGE_USER}@rubyforge.org", "/var/www/gforge-projects/#{RUBYFORGE_PROJECT}", "rdoc").upload
end

begin
  require 'rdoc/task'

  # Generate the RDoc documentation
  RDoc::Task.new do |rdoc|
    rdoc.title = "All-purpose Property List manipulation library"
    rdoc.main  = "README.rdoc"

    rdoc.rdoc_dir = 'rdoc'
    rdoc.rdoc_files.include('README.rdoc', 'LICENSE', 'CHANGELOG')
    rdoc.rdoc_files.include('lib/**')

    rdoc.options = [
      '-H', # show hash marks on method names in comments
      '-N', # show line numbers
    ]
  end
rescue LoadError
  $stderr.puts "Could not load rdoc tasks"
end

# Create compressed packages
spec = Gem::Specification.new do |s|
  s.name    = PKG_NAME
  s.version = PKG_VERSION

  s.summary     = "All-purpose Property List manipulation library."
  s.description = <<-EOD
Plist is a library to manipulate Property List files, also known as plists.  It can parse plist files into native Ruby data structures as well as generating new plist files from your Ruby objects.
EOD

  s.authors  = "Ben Bleything, Patrick May, and Albert Wang"
  s.homepage = "http://github.com/albertyw/plist"

  s.rubyforge_project = RUBYFORGE_PROJECT

  s.has_rdoc = true

  s.files      = RELEASE_FILES
  s.test_files = TEST_FILES

  s.autorequire = 'plist'
end

Gem::PackageTask.new(spec) do |p|
  p.gem_spec = spec
  p.need_tar = true
  p.need_zip = true
end
