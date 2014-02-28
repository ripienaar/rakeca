require 'erb'

def log(msg)
  puts ">>> %s" % msg
end

def ask(prompt, env, default="")
    return ENV[env] if ENV.include?(env)

    if default
      print "#{prompt} (#{default}): "
    else
      print "#{prompt}: "
    end

    resp = STDIN.gets.chomp

    resp.empty? ? default : resp
end

def render_template(template, output, scope)
    tmpl = File.read(template)
    erb = ERB.new(tmpl, 0, "<>")
    File.open(output, "w") do |f|
        f.puts erb.result(scope)
    end
end

def has_ca?
  File.exist?("serial") && File.exist?("openssl.root.cnf") && File.exist?("private")
end

desc "Create a new CSR and private key"
task :gencsr do
  abort "Please specify a cert name to generate using CERT=mycert" unless ENV["CERT"]

  sh "openssl genrsa -out #{ENV['CERT']}.key 2048"
  sh "openssl req -out #{ENV['CERT']}.csr -new -key #{ENV['CERT']}.key -config openssl.server.cnf"
end

desc "Remove all *.csr files in the current directory"
task :clean do
  Dir.glob("*.csr").each do |csr|
    log "Removing #{csr}"
    FileUtils.rm csr
  end
end

desc "Revoke a certificate"
task :revoke do
  abort "Please create a CA using 'rake init'" unless has_ca?
  abort "Please specify a cert to revoke using CERT" unless ENV["CERT"]
  abort "Cannot find the certificate '%s' to revoke" % ENV["CERT"] unless File.exist?(ENV["CERT"])

  log "Revoking certificate %s" % ENV["CERT"]

  sh "openssl ca -config openssl.root.cnf -revoke '%s'" % ENV["CERT"]
  Rake::Task["gencrl"].invoke
end

desc "Sign *.csr files"
task :sign do
  abort "Please create a CA using 'rake init'" unless has_ca?

  Dir.glob("*.csr").each do |csr|
    certname = "%s.%s" % [ File.basename(csr, ".csr"), "cert" ]
    log "Signing %s creating %s" % [csr, certname]
    sh "openssl ca -batch -config openssl.root.cnf -in %s -out %s" % [ csr, certname ]
    FileUtils.rm csr if File.exist?(certname)
  end
end

desc "Recreate the certificate revocation list"
task :gencrl do
  abort "Please create a CA using 'rake init'" unless has_ca?
  sh "openssl ca -config openssl.root.cnf -gencrl -out ca_crl.pem"
end

desc "Completely irreversibly destroy the CA"
task :destroy_ca do
  confirm = ask("Type 'yes' to destroy the CA", "", nil)

  FileUtils.rm_rf %w{serial openssl.root.cnf newcerts index crl private ca_crt.pem ca_crl.pem index.attr index.old serial.old index.attr.old} if confirm == "yes"
end

desc "Create a new CA from scratch"
task :init do
  abort "CA has already been created in the current directory" if has_ca?

  # common_name = ask("CA Common Name", "COMMONNAME", "CA")
  # country_name = ask("CA Country Name", "COUNTRYNAME", nil)
  # state = ask("CA State or Province", "STATE", nil)
  # locality = ask("Locality", "LOCALITY", nil)
  # email = ask("Email Address", "EMAIL", nil)
  # crlurl = ask("URL to the CRL", "", nil)
  timestamp = Time.now.strftime("%s")

  [ 'root', 'server' ].each do |config_type|
    render_template("openssl.cnf.erb", "openssl.#{config_type}.cnf", binding)
  end

  %w{crl newcerts private}.each do |dir|
    log "Creating directory %s" % dir
    FileUtils.mkdir dir
  end

  FileUtils.chmod 0700, "private"
  FileUtils.touch "index"

  File.open("serial", "w") {|f| f.puts "01"}

  sh "openssl req -nodes -config openssl.root.cnf -days 1825 -x509 -newkey rsa:2048 -out ca_crt.pem -outform PEM"
end
