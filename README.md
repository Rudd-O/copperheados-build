# Build CopperheadOS in your Jenkins instance

This is a configuration for Jenkins that will build CopperheadOS (AOSP-based
highly-secure Android-like distribution) images for the Pixel XL — and,
with a bit of tweaking, the other Google Nexus and Pixel phones —
based on the CopperheadOS release schedule, complete with fully compliant
secure boot and anti-theft protection.

This build recipe is based on [the official CopperheadOS build instructions](https://copperhead.co/android/docs/building).
In addition to that, this executable recipe will automatically
rekey F-Droid so F-Droid will have the ability to install applications
as a trusted app store on your phone.  Quite excellent!

This build recipe will also build periodically.  If a successful build
has taken place in the past, the pipeline will exit early with a
successful status, so you do not need to worry about wasting CPU,
memory or disk space on repeat builds of the same thing.  The parameters
used to determine whether a build should run to completion are evident
from the pipeline script — check the script out if you want to know
what decides whether a build continues or not.

## How to use it

Install the Jenkins Pipeline plugin on your Jenkins instance.

Copy the directory containing this file, exactly as-is, into the Jenkins
`jobs` folder.  For example, if your Jenkins `jobs` folder path is
`/var/lib/jenkins/jobs`, then you'd copy this folder so that the copy
ends up at `/var/lib/jenkins/jobs/copperheados-build`.  If you'd like,
rename it afterwards, so it isn't named `copperheados-build`.

Inside the newly-created directory, make a copy of the the `config.xml.j2`
file and name it `config.xml`.

Open the `config.xml` file in your favorite text editor and make the
following changes:

1. Find the `PRODUCT_NAME` parameter under `<parameterDefinitions>`,
   then delete the line that says `{{ android_devices[0] | je }}` from
   the list of parameters.  Now sort the list of parameters so that the
   device you plan to build for most often is at the top.
2. Change the values of the `GIT_USER_NAME` and `GIT_USER_EMAIL`
   parameters to choices of your own.
3. Change the values of `RELEASE_UPLOAD_ADDRESS` and
   `RELEASE_DOWNLOAD_ADDRESS` to suit your updates publishing needs.  Note
   that _you are responsible_ for making sure your Jenkins master node
   can SSH into the host named by `RELEASE_UPLOAD_ADDRESS` and can write
   to the folder named by that variable.  See the documentation adjacent
   to the variables themselves for more information.
4. Tune the rest of the parameters to your own liking, in particular the
   parameter for `NUM_CORES` to speed up the build if your build node
   has a lot of RAM and many cores.
5. In the `<triggers>` section, adjust the trigger times you'd like the
   build to run on.

Now note the product name stored in the `PRODUCT_NAME` variable of the
`config.xml` file.  We'll use this shortly.

Create the keys as per
[the official CopperheadOS build instructions](https://copperhead.co/android/docs/building)
 — relevant snippet for one product name quoted here:

--------------------------------------------------------------------------

To generate keys for `marlin` (you should use unique keys per
device variant / product name):

    mkdir keys/marlin
    cd keys/marlin
    ../../development/tools/make_key releasekey '/C=CA/ST=Ontario/L=Toronto/O=CopperheadOS/OU=CopperheadOS/CN=CopperheadOS/emailAddress=copperheados@copperhead.co'
    ../../development/tools/make_key platform '/C=CA/ST=Ontario/L=Toronto/O=CopperheadOS/OU=CopperheadOS/CN=CopperheadOS/emailAddress=copperheados@copperhead.co'
    ../../development/tools/make_key shared '/C=CA/ST=Ontario/L=Toronto/O=CopperheadOS/OU=CopperheadOS/CN=CopperheadOS/emailAddress=copperheados@copperhead.co'
    ../../development/tools/make_key media '/C=CA/ST=Ontario/L=Toronto/O=CopperheadOS/OU=CopperheadOS/CN=CopperheadOS/emailAddress=copperheados@copperhead.co'
    ../../development/tools/make_key verity '/C=CA/ST=Ontario/L=Toronto/O=CopperheadOS/OU=CopperheadOS/CN=CopperheadOS/emailAddress=copperheados@copperhead.co'
    cd ../..

--------------------------------------------------------------------------

Place those keys in the `keys/<PRODUCT_NAME>` folder under the job directory
you created below the Jenkins `jobs` folder.  You must create one set of
keys per device.

Ensure all files and folders under this job directory are owned by Jenkins
and only readable by it.

Ensure your Jenkins instance has at least one build slave with 16 GB RAM
and 200 GB disk space available.  Give that build slave the label `copperhead`.
Alternatively, change the `node(copperhead)` snippet in `config.xml`
to run it on any slave (see the Jenkins Pipeline reference documentation).

Restart or reload your Jenkins instance.

You're now ready to go.  Build!
