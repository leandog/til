# Running SSL with Capybara
The main tricks here are:

* Register a  server with capybara that has ssl parameters
* Capybara needs to have `server_host`, `server_port`, `app_host`, and `server` set.
* Monkey patching Capybara::Server#responsive? to force ssl (there may be other middleware you can add to avoid this)

## Using WEBrick

* WEBrick has an additional wrinkle that it doesn't understand full-chain certs. You have to split the cert into the cert and the chain.

```ruby
require 'rack/handler/webrick'
require 'webrick/https'

Capybara.register_server :ssl do |app, port, host|
  cert_key = "certificates/mysite-key.pem"
  cert_chain = "certificates/mysite-chain.pem"
  cert = "certificates/mysite-cert.pem"

  opts = {
    :Port => port,
    :Host => host,
    :environment => (ENV['RAILS_ENV'] || "test").dup,
    :daemonize => false,
    :debugger => false,
    :AccessLog => [],
    :Logger => WEBrick::Log::new(nil, 0),
    :SSLEnable => true,
    :SSLVerifyClient => OpenSSL::SSL::VERIFY_NONE,
    :SSLPrivateKey => OpenSSL::PKey::RSA.new(File.open(cert_key).read),
    :SSLExtraChainCert => [OpenSSL::X509::Certificate.new(File.open(cert_chain).read)],
    :SSLCertificate => OpenSSL::X509::Certificate.new(File.open(cert).read),
    :SSLCertName => [["US", WEBrick::Utils::getservername]],
  }
  Rack::Handler::WEBrick.run(app, opts)
end

Capybara.server_port = 3001
Capybara.server_host = "my.host.com"
Capybara.app_host = "https://#{Capybara.server_host}:#{Capybara.server_port}"
Capybara.server = :ssl

module Capybara
  class Server
    def responsive?
      return false if @server_thread && @server_thread.join(0)

      http = Net::HTTP.new(host, @port)
      http.use_ssl = true
      http.verify_mode = OpenSSL::SSL::VERIFY_NONE
      res = http.get('/__identify__')

      if res.is_a?(Net::HTTPSuccess) or res.is_a?(Net::HTTPRedirection)
        return res.body == @app.object_id.to_s
      end
    rescue SystemCallError
      return false
    end
  end
end
```


## Using Thin

Unfortunately, thin throws away any rack ssl options that you pass at initialization.  So you specify an additional block that will set them before the server starts.
Unlike WEBrick, thin does understand full-chain certs.

```ruby
require 'rack/handler/thin'

Capybara.register_server :ssl do |app, port, host|
  cert_key = "certificates/mysite-key.pem"
  cert_full_chain = "certificates/mysite-fullchain.pem"
  opts = {
    :Port => port,
    :Host => host,
    :environment => (ENV['RAILS_ENV'] || "test").dup,
    :daemonize => false,
    :debugger => false,
    :AccessLog => [],
  }
  Rack::Handler::Thin.run(app, opts) do |server|
    server.ssl = true
    server.ssl_options = {
      private_key_file: cert_key,
      cert_chain_file: cert_full_chain
    }
  end
end

Capybara.server_port = 3001
Capybara.server_host = "my.host.com"
Capybara.app_host = "https://#{Capybara.server_host}:#{Capybara.server_port}"
Capybara.server = :ssl

module Capybara
  class Server
    def responsive?
      return false if @server_thread && @server_thread.join(0)

      http = Net::HTTP.new(host, @port)
      http.use_ssl = true
      http.verify_mode = OpenSSL::SSL::VERIFY_NONE
      res = http.get('/__identify__')

      if res.is_a?(Net::HTTPSuccess) or res.is_a?(Net::HTTPRedirection)
        return res.body == @app.object_id.to_s
      end
    rescue SystemCallError
      return false
    end
  end
end
```
