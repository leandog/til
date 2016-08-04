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
