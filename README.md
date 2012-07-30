# ghar: chiphogg's fork

First, check out [the excellent original](https://github.com/philips/ghar) by
Brandon Philips <brandon@ifup.org>.

The goals of this fork include:

  1. More automation (particularly for the setup phase)
  2. Proper repository structure for a particular use case: when...
    - ...we want **all** our computers to share the **same setup**, but...
    - ...they **evolved independently** to their present configurations.

The second point needs a little elaboration.

## What's the "right way" to setup multiple computers?

One important requirement: the original dotfiles for *each machine* must be
preserved forever.  Moreover, it needs to be easy to go back and see how a
given machine was set up, before it got "assimilated" to use the global
configuration.

On reflection, I think **orphan branches** are the best way to go.  Each new
computer gets its own branch, whose initial commit reflects its initial state.
This branch gets *merged into* the master branch, and the computer uses the
master branch from then on.

To set up a new machine and preserve its initial state, perform the following
steps (assuming you've already set up a github repo for your dotfiles; let's
call that repo `dotfiles`):

```bash
# Install ghar
mkdir -p ~/tools
cd ~/tools
git clone git://github.com/chiphogg/ghar.git
export PATH=$PATH:`pwd`/ghar/bin/
. `pwd`/ghar/ghar-bash-completion.sh

# Setup dotfiles repo on this computer.
# Ideally, this whole block would be replaced with something like:
# ghar setup MYCOMPUTERNAME git@github.com:chiphogg/dotfiles.git
# Too bad I don't yet know python!
ghar add git@github.com:chiphogg/dotfiles.git
cd ~/tools/ghar/dotfiles  # a bit fragile; fixing python script would be better
ch-ghar_careful-merge MYCOMPUTERNAME
# NOTE: after the previous step, you may need to do some manual merging.
# After everything has been merged appropriately, it's OK to commit:
git commit

# Your dotfiles are up-to-date!  Install them, and push everything to github.
ghar install
git push --all
```

Note that it's not enough to do this only on the initial setup!  If you add dotfiles from other computers to the master branch, they could easily clobber the corresponding files on all other machines.  What we *really* need to do every time we update is:

  1. `git-fetch` from master branch
  2. Check if master contains any **newly versioned local files**: i.e., files
     which were not previously in the repository, but which have *non-versioned
     __local__ copies*.
  3. If not, merge as usual. But if such files **do** exist, we have to be
     careful; perhaps doing something like the following:
     1. Update the named branch to reflect the **previous** state of master
        (i.e., the last commit before these new files became versioned)
     2. Add the local copies of every newly-versioned file to the **named**
        branch, and commit
     3. Merge the named branch back into master

# Caveat!

This is **highly experimental**!  It kinda sorta works for me, with some manual playing around. The main reason to put it up is so that others can read about my ideas and, if they look any good, they can implement them better than I did.
