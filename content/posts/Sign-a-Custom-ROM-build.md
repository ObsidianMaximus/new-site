+++
title = 'Sign a Custom ROM Build'
date = '2024-05-26T21:05:20+05:30'
tags = [ "guide" , "android" , "rom" ]
showtoc = true
+++

This guide will explain how to **sign** a Custom ROM build (typically unofficial builds, as most official ROMs are now shipping their own private keys).

I will use the [Lineage wiki guide](https://wiki.lineageos.org/signing_builds), but simplify it in my own words at places where I feel this guide did not give more practical information to a new comer like me.

#### The steps to generate the keys are as follows:

**NOTE**: You only need to run this once. If you ever rerun these, you’ll need to migrate between builds.

1. Go to the directory where all yours rom files are originally synced (the root of your custom rom files). Now open a terminal and issue this command: 

    **NOTE**: Please note this beforehand that running the following code will ask you to enter password for several keys and several number of times. I will say that just enter 1 password and copy+paste it on all the places where it asks for a password to be entered. 

     **_DO NOTE THIS PASSWORD DOWN, as we will have to use it when we sign the builds._**

    ```bash
    subject='/C=US/ST=California/L=Mountain View/O=Android/OU=Android/CN=Android/emailAddress=android@android.com'
    mkdir ~/.android-certs
    for cert in bluetooth cyngn-app media networkstack platform releasekey sdk_sandbox shared testcert testkey verity; do \
        ./development/tools/make_key ~/.android-certs/$cert "$subject"; \
    done
    ```

2. Now we are required to re-sign APEX keys. To do so, simply run the following:

    ```bash
    cp ./development/tools/make_key ~/.android-certs/
    sed -i 's|2048|4096|g' ~/.android-certs/make_key
    ```

3. Now finally, we must generate our keys. I will strongly recommend you to generate them **with** a password (re-use the password you used in step 1).

    The command to run is in [here](https://wiki.lineageos.org/signing_builds#generate-keys-with-a-password)


    **NOTE**: If you have issues following up with these steps, I have a script that will execute the commands for you (of course, you have to manually enter the password for each keys). You can find that script [here](https://github.com/ObsidianMaximus/scripts/blob/master/signing/Generate_Keys.sh)
   
    Usage (copy paste this in terminal): 

    ```bash
    curl -O https://raw.githubusercontent.com/ObsidianMaximus/scripts/master/signing/Generate_Keys.sh
    bash Generate_Keys.sh
    ```

#### Now, the steps to **make a .zip flashable package** are as follows:

1. Source the envsetup.sh by executing this command:

    ```bash
    source build/envsetup.sh
    ```

2. Now finally, we will have to start compiling the files for our custom ROM (this will take a long time, depending on your computer's specs). To do so, run the following command (replacing the codename with your device codename):

    ```bash
    breakfast <codename>
    mka target-files-package otatools
    ```

3. Now, it is time to sign all of our APKs and APEXes and build the .zip package.
    
    We will use 2 environment variables to ease our task, as they will allow us to sign without having to enter the password several times. For that, we must issue these 2 commands in the terminal:
    
    ```bash
    export ANDROID_PW_FILE=/path/to/your/password/file
    export EDITOR=your_preferred_text_editor
    ```
    **NOTE**: _Replace the fields above as required. For the **ANDROID_PW_FILE**, just create a file somewhere and input its path there. As for the **EDITOR** variable, choose whichever editor you like (eg. vim, nano, codium, etc)._

    Next run the following commands to call the script to start signing: 

    ```bash
    curl -O https://raw.githubusercontent.com/ObsidianMaximus/scripts/master/signing/Sign.sh
    bash Sign.sh
    ```
    **NOTE**: _This script will invoke a command "**sign_target_files_apks**", which should open the editor which you specified above, but if it does not, then open the file that you had given above in **"ANDROID_PW_FILE"** and you should have stuff inside of it with empty spaces in these "[[[      ]]]" brackets._

    Fill the password in this empty space for all the apks [make sure to fill it between the 3 left and 3 right brackets], and then save this file, and re-run the above script by simple doing: **```bash Sign.sh```**.

    **NOTE**: Wait patiently, it can take some time to do all of the above process.

Congratulations, you have successfully signed the build. Now in case you are wondering where the .zip file is (no, it is not in the usual out/ directory), the .zip is right in the android root tree, where you issued the commands from.

Just clean flash the **signed-ota_update.zip** and device integrity should be met now. 

***

**Update:** Just confirmed that dirty flashing this build over the previous unsigned ROM also works.