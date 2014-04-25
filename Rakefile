# encoding: UTF-8

require 'rake/clean'

namespace :ebook do
        desc "generate epub/mobi/azw/pdf"
        task :generate do
                system("FORMAT=epub ./makeebooks ja")
                system("FORMAT=mobi ./makeebooks ja")
                system("FORMAT=azw3 ./makeebooks ja")
                system("bash makepdfs ja")
        end
end

class StderrDecorator
  def initialize(out)
    @out = out
  end

  def <<(x)
    @out << "#{x}"
    if x.match /REXML/
      raise ""
    end
  end
end

def test_lang(lang, out)
  error_code = false
  chapter_figure = {
    "01-introduction"       => 7,
    "02-git-basics"         => 2,
    "03-git-branching"      => 39,
    "04-git-server"         => 15,
    "05-distributed-git"    => 27,
    "06-git-tools"          => 1,
    "07-customizing-git"    => 3,
    "08-git-and-other-scms" => 0,
    "09-git-internals"      => 4}
  source_files = FileList.new(File.join(lang, '0*', '*.markdown')).sort
  source_files.each do |mk_filename|
    src_file = File.open(mk_filename, 'r')
    figure_count = 0
    until src_file.eof?
      line = src_file.readline
      matches = line.match /^#/
      if matches
        if line.match /^(#+).*#[[:blank:]]+$/
          out<< "\nBadly formatted title in #{mk_filename}: #{line}\n"
          error_code = true
        end
      end
      if line.match /^\s*Insert\s(.*)/
        figure_count = figure_count + 1
      end
    end
    # This extraction is a bit contorted, because the pl translation renamed
    # the files, so the match is done on the directories.
    tab_fig_count = chapter_figure[File.basename(File.dirname(mk_filename))]
    expected_figure_count = tab_fig_count ? tab_fig_count:0
    if figure_count > expected_figure_count
      out << "\nToo many figures declared in #{mk_filename}\n"
      error_code = true
    end
  end
  begin
    mark = (source_files.map{|mk_filename| File.open(mk_filename, 'r'){
                |mk| mk.read.encode("UTF-8")}}).join("\n\n")
    require 'maruku'
    code = Maruku.new(mark, :on_error => :raise, :error_stream => StderrDecorator.new(out))
  rescue
    print $!
    error_code = true
  end
  error_code
end

$out = $stdout

namespace :ci do
  desc "Parallel Continuous integration"
  task :parallel_check do
    require 'parallel'
    langs = FileList.new('??')+FileList.new('??-??')
    results = Parallel.map(langs) do |lang|
      error_code = test_lang(lang, $out)
      if error_code
        print "processing #{lang} KO\n"
      else
        print "processing #{lang} OK\n"
      end
      error_code
    end
    fail "At least one language conversion failed" if results.any?
  end

  (FileList.new('??')+FileList.new('??-??')).each do |lang|
    desc "testing " + lang
    task (lang+"_check").to_sym do
      error_code = test_lang(lang, $out)
      fail "processing #{lang} KO\n" if error_code
      print "processing #{lang} OK\n"
      end
  end

  desc "Continuous Integration"   
  task :check do
    require 'maruku'
    langs = FileList.new('??')+FileList.new('??-??')
    if ENV['debug'] && $lang
      langs = [$lang]
    else
      excluded_langs = [
        ]
      excluded_langs.each do |lang|
        puts "excluding #{lang}: known to fail"
      end
      langs -= excluded_langs
    end
    errors = langs.map do |lang|
      print "processing #{lang} "
      error_code=test_lang(lang, $out)
      if error_code
        print "KO\n"
      else
        print "OK\n"
      end
      error_code
    end
    fail "At least one language conversion failed" if errors.any?
  end
end
