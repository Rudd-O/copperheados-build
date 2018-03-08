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

## How to use it

Install the Jenkins Pipeline plugin on your Jenkins instance.

Copy the directory containing this file, exactly as-is, into the Jenkins
`jobs` folder.  For example, if your Jenkins `jobs` folder path is
`/var/lib/jenkins/jobs`, then you'd copy this folder so that the copy
ends up at `/var/lib/jenkins/jobs/copperheados-build`.  If you'd like,
rename it afterwards, so it isn't named `copperheados-build`.

Inside the newly-created directory, make a copy of the the `config.xml.j2`
file and name it `config.xml`.

Open the `config.xml` file and look for the string `{{`.  Substitute your
preferred values for every instance of `{{ ... }}` that you find there.
This will configure your build to act the way you want it to.

Now note the product name stored in the `PRODUCT_NAME` variable of the
`config.xml` file.  We'll use this shortly.

Create the keys as per the build instructions — relevant snippet quoted here:

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

Place those keys in the `keys/<product name>` folder under the job directory
you created below the Jenkins `jobs` folder.

Ensure all files and folders under this job directory are owned by Jenkins
and only readable by it.

Ensure your Jenkins instance has at least one build slave with 16 GB RAM
and 200 GB disk space available.  Give that build slave the label `copperhead`.
Alternatively, change the `node(copperhead)` snippet in `config.xml`
to run it on any slave (see the Jenkins Pipeline reference documentation).

Restart or reload your Jenkins instance.

You're now ready to go.  Build!
