task default: :prepare

desc 'Builds the static files'
task :build do
  puts `jekyll build`
  puts '[*] Built files'
end

desc 'Minifies the stylesheets'
task :minify_css do
  `node_modules/.bin/cleancss -o _deploy/assets/m.css _source/assets/m.css`
  puts '[*] Minified CSS'
end

desc 'Builds the static files and minifies the stylesheets'
task prepare: [:build, :minify_css]

desc 'Deploy the built site'
task :deploy do
  system File.read('deploy')
  puts '[*] Deployed'
end

desc 'Build, minify and deploy'
task all: [:prepare, :deploy]
