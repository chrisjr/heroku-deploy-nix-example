This is an example for Nix deployment to Heroku using the
[heroku-buildpack-nix-proot](http://github.com/chrisjr/heroku-buildpack-nix-proot)
buildpack. See that buildpack for more general instructions.

This repo also includes a Vagrantfile for local testing on a similar environment
to the deprecated "cedar" stack. This has the benefit that a binary closure
will be generated on your local machine with no time limit; this closure
can be downloaded by Heroku without having to run a one-off build dyno.

Create your `secrets` file with the S3 credentials like this:
```
NIX_S3_KEY=...
NIX_S3_SECRET=...
NIX_S3_BUCKET=...
```
## Deployment

### Vagrant
The first time, clone heroku-buildpack-nix-proot into the directory above:
```bash
pushd ..
git clone https://github.com/chrisjr/heroku-buildpack-nix-proot.git
popd
```

(You can put the buildpack wherever you like, just edit the Vagrantfile's
synced folder accordingly.)

To test using Vagrant:
```bash
vagrant up
vagrant ssh

# on virtual machine
bash /vagrant/do_build
```

The BUILD_DIR (contents of the slug) will be copied to the 'build' directory,
so you can examine it from the outside OS.

If the VM's already running and you'd like to try a change, replace `vagrant up` with 
`vagrant reload --provision`.

### Heroku
To deploy directly to Heroku:
```bash
source secrets
heroku create -b http://github.com/chrisjr/heroku-buildpack-nix-proot
heroku config:set NIX_S3_KEY=$NIX_S3_KEY \
                  NIX_S3_SECRET=$NIX_S3_SECRET \
                  NIX_S3_BUCKET=$NIX_S3_BUCKET
git push heroku master
# wait for initial Nix install...

heroku run build
# this actually builds the closure and uploads it to S3
# for a larger app, you'd need `heroku run -s PX build`

# final deploy
git commit --amend --no-edit
git push -f heroku master
```

Since this app is small, you can also run `heroku config:set NIX_BUILD_ON_PUSH=1`
and only do the first `git push heroku master`.
