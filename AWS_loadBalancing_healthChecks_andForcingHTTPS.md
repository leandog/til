# AWS, load balancing, health checks, and forcing HTTPS

Do you have an app deployed to an Elastic Beanstalk environment? Do you need to re-route all requests over HTTP through HTTPS? Do you want to maintain the ability for AWS to track the environment's "health" (which it does by pinging an endpoint over HTTP, *not* HTTPS)?

If so, this post is for you.

(Understanding the environment may be of some help in picturing the eventual solution. See footnote #1.)

The solution is...but I ~~cheated~~ am standing on the shoulders of giants. The solution is here: see [this article](https://medium.com/trisfera/getting-to-know-and-love-aws-elastic-beanstalk-configuration-files-ebextensions-9a4502a26e3c), especially sections 1 & 2. The main points are:

1. AWS will generate a default Nginx config that includes another file that contains the configuration for your application.
2. You don't want to include that other file (or you want to override its effect)
3. So, create your own config file, included in the `.ebextensions` folder in the root of your project, with a special name and with special directives so that it will be included in the general Nginx config file
4. Be sure to include a block in your version of the config file that will actually have the logic to check the URL and redirect if needed. It should also check if it's a request for the "health check" endpoint and let it through:

```
set $should_enforce_https 0;
if ($http_x_forwarded_proto != 'https') {
  set $should_enforce_https 1;
}
if ($request_uri ~ ^/path/to/health/check$) {
  set $should_enforce_https 0;
}
if ($should_enforce_https = 1) {
  return 301 https://$host$request_uri;
}
```

Note that Nginx doesn't allow nested `if`-statements, hence the weird use of a flag and multiple sequential conditions.

I hoped there would be a configuration one could enable on the load balancer itself to do this, but there's not currently (Spring '17) a way to do that. This works because the Nginx proxy intercepts traffic after the load balancer but before the application code.

---

Footnotes:

1. Some ascii art:

   - the Elastic Load Balancer (elb) directs traffic to the web nodes, which are EC2 instances; the certificate is installed on this server so that all encrypted traffic here; it is also worth noting that it must be configured to accept traffic at both port 443 *and* port 80, then forward that traffic to port 80 on the web nodes

```
   +---+         +---+
   | L |---------| W |
   | O |         | S |
   | A |         | 1 |
   | D |         +---+
   |   |         (ec2)
   | B |
   | A |------    ...
   | L |
   | A |
   | N |         +---+
   | C |---------| W |
   | E |         | S |
   | R |         | N |
   +---+         +---+
   (elb)         (ec2)
```

   - our set-up on each web node was the standard Elastic Beanstalk Linux/Ruby-on-Rails image, which has this high-level structure

```
       (EC2 Instance)
   +---+---+-------------+
   |   |   |             |
   | n |   |             |
   | g | p |             |
   | i | u |   RoR/app   |
   | n | m |             |
   | x | a |             |
   |   |   |             |
   +---+---+-------------+
     ^   ^      ^
     |   |      |
     |   |      +------ the application code
     |   |
     |   +------------- configures unix socket, launches ruby app workers
     |
     +----------------- listens on port 80, then forwards to the unix socket Puma/Ruby's running
```

   - the Nginx instance on this server functions as a proxy to shuttle traffic from the load balancer to your application
