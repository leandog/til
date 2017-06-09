# Super easy SSL with Puma

While very basic, and not totally documented, Puma allows you to add SSL to your application server very easily. Under the covers it wraps OpenSSL in a small library called MiniSSL.

```bash
$ puma -b 'ssl://0.0.0.0:443?key=path_to_key&cert=path_to_cert'
```

## Adding the certificate authority chain

Unlike nginx, Puma doesn't seem to like having the ca certificate concatenated with the application certificate. Instead, you need to pass an undocumented argument to Puma specifying where the chain cert resides.

```bash
$ puma -b 'ssl://0.0.0.0:443?key=path_to_key&cert=path_to_cert&verify_mode=none&ca=path_to_chain.crt'
```

Easy as pie!


## Puma on MacOS Sierra (or later) "SSL not available in this build"

Newer versions of MacOS do not ship with the proper SSL headers  (https://github.com/puma/puma/issues/971).

To get around this, install OpenSSL and recompile the gem with the correct headers.  If your bundle is using puma 3.7.0 for instance:

```bash
brew install openssl
gem uninstall puma
gem install puma -v '3.7.0' -- --with-cppflags=-I/usr/local/opt/openssl/include --with-ldflags=-L/usr/local/opt/openssl/lib
```

Then you can run the `puma -b` command as before.
