load 'deploy' if respond_to?(:namespace) # cap2 differentiator
Dir['vendor/plugins/*/recipes/*.rb'].each { |plugin| load(plugin) }

load 'config/deploy' # remove this line to skip loading any of the default tasks

after "deploy:update", :copy_database_yml
after "deploy:update", :set_permissions
after "deploy:update", :update_sphinx
after "deploy:setup", :create_database

task :create_database, :role => [:web, :app] do
  run "echo 'create database clojuredocs_production' | mysql -u#{db_user}"
end

task :copy_database_yml, :role => [:web, :app] do
  run "cp #{release_path}/config/database.example.yml #{release_path}/config/database.yml"
end

task :set_permissions, :role => [:web, :app] do
  run "chown -R nobody #{release_path}/tmp"
  run "chown -R nobody #{release_path}/public"
  run "touch #{release_path}/log/production.log"
  run "chmod 0666 #{release_path}/log/production.log"
end

namespace :bundler do
  task :create_symlink, :roles => :app do
    shared_dir = File.join(shared_path, 'bundle')
    release_dir = File.join(current_release, '.bundle')
    run("mkdir -p #{shared_dir} && ln -s #{shared_dir} #{release_dir}")
  end
 
  task :bundle_new_release, :roles => :app do
    bundler.create_symlink
    run "cd #{release_path} && /opt/ruby-enterprise/bin/bundle install --without test"
  end
end
 
after 'deploy:update_code', 'bundler:bundle_new_release'

task :update_sphinx, :role => [:web, :app] do
  run("killall searchd; true")
  run("cd #{deploy_to}/current && #{rake} thinking_sphinx:rebuild RAILS_ENV=production")
end