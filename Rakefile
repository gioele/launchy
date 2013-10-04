require 'bundler/gem_tasks'

require 'rake/clean'

require 'rake/testtask'
Rake::TestTask.new( :test ) do |t|
  t.libs         = %w[ lib spec ]
  t.pattern      = "spec/**/*_spec.rb"
end

task :test_requirements
task :test => :test_requirements
task :default => :test

require 'rdoc/task'

require 'simplecov'
desc 'Run tests with code coverage'
task :coverage do
  ENV['COVERAGE'] = 'true'
  Rake::Task[:test].execute
end


#------------------------------------------------------------------------------
# Fixme - look for fixmes and report them
#------------------------------------------------------------------------------
namespace :fixme do
  task :default => 'manifest:check' do
    This.manifest.each do |file|
      next if file == __FILE__
      next unless file =~ %r/(txt|rb|md|rdoc|css|html|xml|css)\Z/
      puts "FIXME: Rename #{file}" if file =~ /fixme/i
      IO.readlines( file ).each_with_index do |line, idx|
        prefix = "FIXME: #{file}:#{idx+1}".ljust(42)
        puts "#{prefix} => #{line.strip}" if line =~ /fixme/i
      end
    end
  end

  def fixme_project_root
    This.project_path( '../fixme' )
  end

  def fixme_project_path( subtree )
    fixme_project_root.join( subtree )
  end

  def local_fixme_files
    This.manifest.select { |p| p =~ %r|^tasks/| }
  end

  def outdated_fixme_files
    local_fixme_files.reject do |local|
      upstream     = fixme_project_path( local )
      Digest::SHA256.file( local ) == Digest::SHA256.file( upstream )
    end
  end

  def fixme_up_to_date?
    outdated_fixme_files.empty?
  end

  desc "See if the fixme tools are outdated"
  task :outdated => :release_check do
    if fixme_up_to_date? then
      puts "Fixme files are up to date."
    else
      outdated_fixme_files.each do |f|
        puts "#{f} is outdated"
      end
    end
  end

  desc "Update outdated fixme files"
  task :update => :release_check do
    if fixme_up_to_date? then
      puts "Fixme files are already up to date."
    else
      puts "Updating fixme files:"
      outdated_fixme_files.each do |local|
        upstream = fixme_project_path( local )
        puts "  * #{local}"
        FileUtils.cp( upstream, local )
      end
      puts "Use your git commands as appropriate."
    end
  end
end
desc "Look for fixmes and report them"
task :fixme => "fixme:default"

#------------------------------------------------------------------------------
# Release - the steps we go through to do a final release, this is pulled from
#           a compbination of mojombo's rakegem, hoe and hoe-git
#
# 1) make sure we are on the master branch
# 2) make sure there are no uncommitted items
# 3) check the manifest and make sure all looks good
# 4) build the gem
# 5) do an empty commit to have the commit message of the version
# 6) tag that commit as the version
# 7) push master
# 8) push the tag
# 7) pus the gem
#------------------------------------------------------------------------------
task :release_check do
  unless `git branch` =~ /^\* master$/
    abort "You must be on the master branch to release!"
  end
  unless `git status` =~ /^nothing to commit/m
    abort "Nope, sorry, you have unfinished business"
  end
end

__END__
desc "Create tag v#{This.version}, build and push #{This.platform_gemspec.full_name} to rubygems.org"
task :release => [ :release_check, 'manifest:check', :gem ] do
  sh "git commit --allow-empty -a -m 'Release #{This.version}'"
  sh "git tag -a -m 'v#{This.version}' v#{This.version}"
  sh "git push origin master"
  sh "git push origin v#{This.version}"
  sh "gem push pkg/#{This.platform_gemspec.full_name}.gem"
end
