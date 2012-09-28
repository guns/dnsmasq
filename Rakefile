# Copyright (c) 2011 Sung Pae <sung@metablu.com>
# Distributed under the MIT license.
# http://www.opensource.org/licenses/mit-license.php

require 'yaml'
require 'erb'

@config = 'config.yml'

def execute env, cmd
  puts (env.map { |k,v| "#{k}=#{v.inspect}" } + cmd).join(' ')
  system env, *cmd
end

task :default => :build

task :env do
  @env = YAML.load_file @config
end

desc 'Clean build droppings'
task :clean do
  execute Hash.new, %w[make clean]
  rm_f @config
end

desc 'Configure and build dnsmasq (with i18n)'
task :build do
  # Build with DNS features only
  env = {}
  env['PREFIX'] = ENV['PREFIX'] || '/opt/dnsmasq'
  env['COPTS' ] = %Q(-DNO_TFTP -DNO_DHCP -DHAVE_IDN -DCONFFILE='"#{env['PREFIX']}/etc/dnsmasq.conf"')

  # Some extra flags for OS X
  if RUBY_PLATFORM[/darwin/]
    env['LDFLAGS'] = '-lintl'

    # Homebrew linking help
    if system '/bin/sh -c "command -v brew" &>/dev/null'
      gettext = %x(brew --prefix gettext).chomp
      env['PATH'   ]  = gettext + '/bin:' + ENV['PATH']
      env['COPTS'  ] << %Q( -I#{gettext}/include )
      env['LDFLAGS'] << %Q( -L#{gettext}/lib )
    end
  else
    env['LDFLAGS'] = '-lidn'
  end

  # Clean and build!
  Rake::Task[:clean].invoke
  execute env, %W[make --jobs=#{(ENV['JOBS'] || 2).to_i} all-i18n]

  # Store configuration values for other programs / tasks
  puts "Writing configuration to #{@config.inspect}"
  File.open(@config, 'w') { |f| f.puts env.to_yaml }
  puts env.to_yaml
end

desc 'Install dnsmasq'
task :install => :env do
  execute @env, %w[make install-i18n]

  @prefix = @env['PREFIX']
  confdir = File.join @prefix, 'etc'
  mkdir_p confdir

  # Install configuration file
  confbuf = ERB.new(File.read 'contrib/guns/dnsmasq.conf.erb').result binding
  conf = confdir + '/dnsmasq.conf'
  unless File.exists? conf
    puts "installing #{conf}"
    File.open(conf, 'w') { |f| f.write confbuf }
  end

  # Along with a default copy
  defaultrc = confdir + '/dnsmasq.conf.default'
  puts "installing #{defaultrc}"
  File.open(defaultrc, 'w') { |f| f.write confbuf }

  # Install local resolv.conf, hosts file, and rakefile
  Dir['contrib/guns/{hosts,resolv.conf,Rakefile}'].each do |f|
    dst = File.join confdir, File.basename(f)
    cp f, dst unless File.exists? dst
  end

  Rake::Task['install:init'].execute
  Rake::Task['install:service'].execute
end

namespace :install do
  desc 'Install init file'
  task :init => :env do
    @prefix = @env['PREFIX']
    rc = File.join @prefix, 'etc/rc.d/dnsmasq'
    FileUtils.mkdir_p File.dirname(rc)
    puts "Installing #{rc}"
    File.open rc, 'w', 0755 do |f|
      f.puts ERB.new(File.read 'contrib/guns/dnsmasq.rc.erb').result(binding)
    end
  end

  desc 'Install systemd service file'
  task :service => :env do
    @prefix = @env['PREFIX']
    service = File.join @prefix, 'lib/systemd/system/dnsmasq.service'
    puts "Installing #{service}"
    FileUtils.mkdir_p File.dirname(service)
    File.open service, 'w', 0644 do |f|
      f.puts ERB.new(File.read 'contrib/guns/dnsmasq.service.erb').result(binding)
    end
  end
end
