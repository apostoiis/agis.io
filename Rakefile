task default: :prepare

desc 'Builds the static files'
task :build do
  puts `jekyll build`
  puts '[*] Built files'
end

desc 'Minify CSS'
task :minify do
  `node_modules/.bin/cleancss -o _site/css/main.css css/main.css`
  puts '[*] Minified CSS'
end

desc 'Builds the static files and minifies the stylesheets'
task prepare: [:build, :minify]
