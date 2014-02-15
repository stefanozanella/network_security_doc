root_file = "index.tex"
temp_dir = "tmp"
pdf_temp = "#{temp_dir}/pdf"
pdf_out = "pdf"
html_temp = "#{temp_dir}/html"
html_out = "html"

desc "LaTeX -> HTML + PDF"
task :build => [:pdf, :html]

desc "Generate PDF"
task :pdf do
  safe_mkdir pdf_out, pdf_temp

  sh %Q{pdflatex -output-directory=#{pdf_temp} #{root_file}}

  mv FileList["#{pdf_temp}/*.pdf"], pdf_out
end

desc "Generate HTML5"
task :html do
  safe_mkdir html_out, html_temp

  sh %Q{htlatex #{root_file} "html5,charset=utf-8,-css" "" "" "-output-directory=#{html_temp}"}

  # This is done mimicking what `htlatex` does, only handling the fact that
  # tex and auxiliary files live in different places (which is not supported
  # out-of-the-box by htlatex itself
  cd html_temp do
    sh %Q{tex4ht -f/../../#{root_file} -i~/tex4ht.dir/texmf/tex4ht/ht-fonts/ -cunihft -utf8}
    sh %Q{t4ht -f/../../#{root_file}}
  end

  mv FileList["#{html_temp}/*.html"], html_out
end

desc "Clean all intermediate files"
task :clobber do
  rm_rf FileList["#{temp_dir}/**/*"]
end

desc "Clean all output files, artifacts included"
task :clean => :clobber do
  rm_rf pdf_out
  rm_rf html_out
end

def safe_mkdir(*dirs)
  [dirs].flatten.each do |dir|
    mkdir_p dir unless Dir.exists? dir
  end
end
