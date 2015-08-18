# pushing to heroku

Imagine this setup:

2 heroku instances: 1 for staging, 1 for production

2 branches: 1 for staging, 1 for production

I want to push the staging branch to the staging heroku.  So I run:

```bash
git push heroku-staging staging
```

But it doesn't actually "do" a deployment.  The code on the running server seems unchanged.
Heroku by default wants you to use master for deployments.  Which is ok, but limiting.

Here's how you push whatever branch you want to heroku:
```bash
git push heroku-staging staging:master
```

`heroku-staging` is my remote
`staging` is the branch I want to push
`master` is what my remote calls this branch

Success!
