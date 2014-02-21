# Tasks configuration
ROOT_FILE = "index.tex"
TEMP_DIR = "tmp"
PDF_TEMP = "#{TEMP_DIR}/pdf"
PDF_OUT = "pdf"
HTML_TEMP = "#{TEMP_DIR}/html"
HTML_OUT = "html"
GH_PAGES_BRANCH = "gh-pages"
PUBLISH_DIR = "gh-pages"

ENV['TEXINPUTS'] = ".:packages:#{ENV['TEXINPUTS']}"

desc "LaTeX -> HTML + PDF"
task :build => [:pdf, :html]

desc "Generate PDF"
task :pdf do
  safe_mkdir PDF_OUT, PDF_TEMP

  sh %Q{pdflatex -output-directory=#{PDF_TEMP} #{ROOT_FILE}}

  mv FileList["#{PDF_TEMP}/*.pdf"], PDF_OUT
end

desc "Generate HTML5"
task :html do
  safe_mkdir HTML_OUT, HTML_TEMP

  sh %Q{htlatex #{ROOT_FILE} "html5,charset=utf-8,-css" "" "" "-output-directory=#{HTML_TEMP}"}

  # This is done mimicking what `htlatex` does, only handling the fact that
  # tex and auxiliary files live in different places (which is not supported
  # out-of-the-box by htlatex itself
  cd HTML_TEMP do
    sh %Q{tex4ht -f/../../#{ROOT_FILE} -i~/tex4ht.dir/texmf/tex4ht/ht-fonts/ -cunihft -utf8}
    sh %Q{t4ht -f/../../#{ROOT_FILE}}
  end

  mv FileList["#{HTML_TEMP}/*.html"], HTML_OUT
end

desc "Publishes current version of the project to Github Pages"
task :publish => :build do
  if working_dir_is_dirty?
    puts "Cannot publish: you have unstaged changes in your working directory. Please commit or stash them before proceeding."
    show_working_dir_status
    exit 1
  end

  if index_is_dirty?
    puts "Cannot publish: your index contains uncommitted changes. Please commit or stash them before proceeding."
    show_index_status
    exit 1
  end

  if not Dir.exists? PUBLISH_DIR
    FileUtils.mkdir_p PUBLISH_DIR
    sh "git clone #{remote_url} -b #{GH_PAGES_BRANCH} #{PUBLISH_DIR}"
  else
    puts "Pulling changes from #{GH_PAGES_BRANCH}"
    FileUtils.cd PUBLISH_DIR do
      sh "git pull origin #{GH_PAGES_BRANCH}"
    end
  end

  puts "Copying updated content"
  Dir.glob("#{HTML_OUT}/*") do |path|
    FileUtils.cp_r path, PUBLISH_DIR
  end

  FileUtils.cd PUBLISH_DIR do
    puts "Publishing documentation to Github Pages..."
    sh "git add ."
#    sh "git commit -m 'Update to #{last_commit_sha}.'"
#    sh "git push origin #{GH_PAGES_BRANCH}"
    puts "Done."
  end
end

desc "Clean all intermediate files"
task :clobber do
  rm_rf FileList["#{TEMP_DIR}/**/*"]
end

desc "Clean all output files, artifacts included"
task :clean => :clobber do
  rm_rf PDF_OUT
  rm_rf HTML_OUT
end

def safe_mkdir(*dirs)
  [dirs].flatten.each do |dir|
    mkdir_p dir unless Dir.exists? dir
  end
end

def working_dir_is_dirty?
  not system "git diff-files --quiet --ignore-submodules --"
end

def index_is_dirty?
  not system "git diff-index --cached --quiet HEAD --ignore-submodules --"
end

def show_working_dir_status
  %x{git diff-files --name-status -r --ignore-submodules -- >&2}
end

def show_index_status
  %x{git diff-index --cached --name-status -r --ignore-submodules HEAD -- >&2}
end

def last_commit_sha
  %x{git log --oneline -n 1 | awk '{print $1}'}.strip
end

def remote_url
  %x{git config --get remote.origin.url}.strip
end
