####NAME

buildbox - easy-to-use Linux sandbox with version control

####SYNOPSIS

buildbox -P PROFILE OPTIONS

buildbox -r

####DESCRIPTION
buildbox utility is a wrapper around chroot and git to facilitate creation & use of sandboxed
Linux environment (called "buildbox").
Sandboxed Linux is based on Ubuntu 12.04 LTS and is stored in git repository.
By default interactive shell is run in buildbox, but one may execute non-interactive commands
also.

Sandboxed Linux is setup in dedicated directory derived from PROFILE name.
Refer to PROFILES section for more information on buildbox's profiles.

####OPTIONS
#####-P PROFILE

  Setup buildbox for profile PROFILE. If profile is not existing then -s option is obligatory.
  Refer to PROFILES section for more information on profiles.

#####-s VARIANT

  Use specific VARIANT of buildbox during setup.
  If PROFILE is already existing (setup previously) then this option is ignored.
  Refer to -r option to get list of supported variants and -n to create new variant of
  buildbox.

  Delete buildbox. Local deletion only, does not affect git repository.
  It unmounts all filesystems, reverts root permissions from several files and deletes
  local copy of buildbox.

  See EXTENDED EXAMPLES section to see how to remove buildbox permanently from git
  repository.

#####-n SRC_VARIANT

  Create new buildbox for variant VARIANT using SRC_VARIANT as a base.
  If unsure, use master for SRC_VARIANT to get new default buildbox.
  Updates list of variants in git repository. If VARIANT already exists in repository then
  this options is ignored.


  List available buildboxes in git repository.

#####-m BUILDBOX_DIR=HOST_DIR
  Create BUILDBOX_DIR directory within buildbox and map it to HOST_DIR from host
  filesystem.

  Multiple -m options may be given to map multiple directories.

  When specifying BUILDBOX_DIR one may use special token BUILDBOX_HOME that represents
  user's home directory withing buildbox.


  List all changes in buildbox.

#####-V VER

  Use version VER of buildbox. VER must be a git sha1 as displayed by -l option. Can not
  be used with -U option.

  Note:
  All changed files within buildbox will be discarded.
  Not tracked files (including user's home directory) will be retained.

  Update buildbox to most recent version available. Can not be used with -V VER option.

  Note:
  All changed files within buildbox will be discarded.
  Not tracked files (including user's home directory) will be retained.

  buildbox may be updated, only if there are no other buildbox sessions (interactive or
  non-interactive) using given buildbox.

  Be verbose - displays each command executed by buildbox.

#####-c CMD

  Execute command CMD in buildbox, instead of switching to interactive shell.

####PROFILES
buildbox is always setup in in dedicated directory:
BUILDBOX_HOME/PROFILE/content-POSTFIX

BUILDBOX_HOME is `$HOME/.buildbox` by default, or any other directory that can be specified as
in `$HOME/.buildbox_dir` file.

POSTFIX is HEAD by default or value VER specified with -V VER parameter.

-U option is always applied to BUILDBOX_HOME/PROFILE/content-HEAD directory.

Additionally all directory mappings (specified with -m parameter) are remembered in
BUILDBOX_HOME/PROFILE/mapping file and are automatically applied if -m is missing in
subsequent buildbox executions.

Moreover special mapping for ccache is always added, namely:
-mBUILDBOX_HOME/.ccache=BUILDBOX_DIR/PROFILE/ccache

####SETUP GIT REPOSITORY
To setup initial git repository use `prepare_repo.sh` script.
Destination directory must be specified as parameter.

Url to resulting bare git repository must be put to buildbox configuration file -
`$HOME/.buildbox_repo`

####SETUP BUILDBOX
In order to run buildbox tool it is required to use modern Ubuntu distribution with git
package installed:

        $ sudo apt-get install git

Since buildbox uses lockfile command for synchronization procmail package must be installed

        $ sudo apt-get install procmail

User running buildbox must be able to execute following commands using
sudo: `mount`, `umount`, `chroot`, `chown` and `chmod`

so add following line to /etc/sudoers, substituting user_name with proper value:

        user_name ALL=NOPASSWD: /bin/mount,/bin/umount,/usr/sbin/chroot,/bin/chown,/bin/chmod

####EXAMPLES
1. List all buildboxes available in git repository:

        $ buildbox -r
        [...]
        master
        [...]

  master is a template environment, other environments are created using
  this as a base one.

2. Setup default buildbox:

        $ buildbox -P example -s master

  When process is completed user is presented with shell.
  Issue 'logout' or press Ctrl+D to leave buildbox.

3. Create new variant (EXAMPLE01) of buildbox:

        $ buildbox -P example -s EXAMPLE01 -n

  After this command is completed your new buildbox is available in git repository. You do
  not have to add/commit/push any files just after creation of new buildbox.

4. Add new dir/file to buildbox of EXAMPLE01 variant:

        $ mkdir -p example/rootfs/opt/new_dir
        $ touch example/rootfs/new_dir/new_file
        $ cd example
        $ git add rootfs/new_dir
        $ git commit -m"some meaningful message"
        $ git push
        $ cd ..

  buildbox for variant EXAMPLE01 is updated at this point.
  History of changes can be examined with:

      $ buildbox -P example -l

5. Execute listing of buildbox root directory instead of running interactive shell:

        $ buildbox -P example -c "ls /"

6. Update buildbox to most recent version:

        $ buildbox -P example -U

7. Use indicated version of buildbox within profile example:

        $ buildbox -P example -V deadbee

  Particular versions of buildbox are kept separately, so:
  /home/foobar/.buildbox/example/content-deadbee will be initialized.

####ADVANCED TOPICS

#####INITIALISATION STEPS

When buildbox is setup following steps are executed:

1. Root filesystem is fetched from git repository to local directory
   (which is BUILDBOX_HOME/PROFILE/content-HEAD),
2. User is created with UID/GID equal to UID/GID of user executing buildbox tool. Name of
   user in buildbox is created by appending name of variant to name of user at host,
3. `/proc`, `/dev`, `/dev/pts`, `/tmp` and `/sys` directories from host filesystem are mounted within
   local directory,
4. Permissions and ownerships of selected files within local directory get changed to allow
   sudo executions within buildbox,
5. `/etc/hosts` file from host filesystem is copied to local directory,
6. All directories supplied to -m parameter are mapped (mounted with bind option) from host
   directories to local directory,
7. `/etc/profile.d/setup_env.sh` file is created in local directory. It contains all env
   variables from host shell running buildbox. Env variables will be replicated within
   buildbox, if not already defined.
   E.g: `PATH` variable is setup in buildbox before execution of
   `setup_env.sh` script, so `PATH` variable will not be overwritten with contents from host
   shell,
8. Configuration files related to date&time settings (`/etc/localtime` & `/etc/timezone`) are
   copied from host to buildbox,
9. Script showing PROFILE directory and VARIANT is installed in `/usr/bin/whereami` of buildbox.

buildbox tool may skip some of above steps if they have been already completed during previous
executions. E.g root filesystem is not fetched if already available locally.

#####EXTENDED EXAMPLES

1. Removing buildbox variant from git repository:

        $ buildbox -P test -s TEST
        $ cd ~/.buildbox/test/
        $ git push origin :TEST
        $ cd ~
        $ buildbox -P test -D

2. Checking modified files in buildbox directory:

        $ cd test/
        $ git status

