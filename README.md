# Serverspec [![Gem Version](https://badge.fury.io/rb/serverspec.png)](http://badge.fury.io/rb/serverspec) [![BuildStatus](https://secure.travis-ci.org/serverspec/serverspec.png)](http://travis-ci.org/serverspec/serverspec) [![Code Climate](https://codeclimate.com/github/serverspec/serverspec.png)](https://codeclimate.com/github/serverspec/serverspec)

RSpec tests for your servers configured by Puppet, Chef or anything else

You can see the details of serverspec on [serverspec.org](http://serverspec.org/).

----

## Quick Installation

Add this line to your application's Gemfile:

    gem 'serverspec'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install serverspec

----

## Usage

```
$ serverspec-init
Select OS type:

  1) UN*X
  2) Windows

Select number: 1

Select a backend type:

  1) SSH
  2) Exec (local)

Select number: 1

Vagrant instance y/n: n
Input target host name: www.example.jp

 + spec/
 + spec/www.example.jp/
 + spec/www.example.jp/httpd_spec.rb
 + spec/spec_helper.rb
 + Rakefile
```

`spec/www.example.jp/httpd_spec.rb` is a sample spec file and its content is like this.

```ruby
require 'spec_helper'

describe package('httpd') do
  it { should be_installed }
end

describe service('httpd') do
  it { should be_enabled }
  it { should be_running }
end

describe port(80) do
  it { should be_listening }
end

describe file('/etc/httpd/conf/httpd.conf') do
  it { should be_file }
  its(:content) { should match /ServerName www.example.jp/ }
end
```

You can write specs for testing servers like this.

Serverspec with SSH backend logs in to target servers as a user configured in ``~/.ssh/config`` or a current user. If
 you'd like to change the user, please edit the below line in ``spec/spec_helper.rb``.

```ruby
user = options[:user] || Etc.getlogin
```

Run tests.

```
$ rake spec
/usr/bin/ruby -S rspec spec/www.example.jp/httpd_spec.rb
......

Finished in 0.99715 seconds
6 examples, 0 failures
```

----
## Multi OS support

Serverspec supports Darwin based OS, Red Hat based OS, Debian based OS, Gentoo and Solaris.

Serverspec can detect target host's OS automatically.

If you'd like to set target host's OS explicitly, you should include `SpecInfra::Helper::OSName` in `spec/spec_helper.rb` like this.


```ruby
require 'serverspec'
require 'pathname'
require 'net/ssh'

include SpecInfra::Helper::Ssh
include SpecInfra::Helper::Debian

RSpec.configure do |c|
  # Add SSH before hook in case you use the SSH backend
  # (not required for the Exec backend)
  c.before do
    host  = File.basename(Pathname.new(example.metadata[:location]).dirname)
    if c.host != host
      c.ssh.close if c.ssh
      c.host  = host
      options = Net::SSH::Config.for(c.host)
      user    = options[:user] || Etc.getlogin
      c.ssh   = Net::SSH.start(c.host, user, options)
    end
  end
end
```

You can select from:

* SpecInfra::Helper::RedHat
* SpecInfra::Helper::Debian
* SpecInfra::Helper::Gentoo
* SpecInfra::Helper::Solaris
* SpecInfra::Helper::Darwin

## Vagrant support

Serverspec now has Vagrant support, and can be automatically configured from a Vagrantfile.

```
$ serverspec-init
Select a backend type:

  1) SSH
  2) Exec (local)

Select number:1

Vagrant instance y/n: y
Auto-configure Vagrant from Vagrantfile? y/n: y
0) web
1) db
Choose a VM from the Vagrantfile: 1
 + spec/db/
 + spec/db/httpd_spec.rb
 + spec/spec_helper.rb
 + Rakefile
```

See details on [serverspec.org](http://serverspec.org)

----

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
