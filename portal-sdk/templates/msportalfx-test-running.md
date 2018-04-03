## Running

* In Dev

    * From VS

    * From cmdline

* CI

    * Cloudtest

      Running mocha nodejs tests in cloudtest requires a bit of engineering work to set up the test VM. Unfortunetly, the nodejs test adaptor cannot be used with vs.console.exe since it requires a full installation of Visual Studio which is absent on the VMs. Luckily, we can run a script to set up our environment and then the Exe Execution type for our TestJob against the powershell/cmd executable.

        * Environment Setup

            Nodejs (and npm) is already installed on the cloudtest VMs. Chrome is not installed by default, so we can include the chrome executable in our build drop for quick installation.

            setup.bat
            ```
            cd UITests
            call npm install --no-optional
            call npm install -g typescript
            call "%APPDATA%\npm\tsc.cmd"
            call chrome_installer.exe /silent /install
            exit 0
            ```

        * Running Tests

            Use the Exe execution type in your TestJob to specify the powershell (or cmd) exe. Then, point to a script which will run your tests:

            TestGroup.xml
            ```
            <TestJob Name="WorkspaceExtension.UITests" Type="SingleBox" Size="Small" Tags="Suite=Suite0">
                <Execution Type="Exe" Path="C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" Args='[SharedWorkingDirectory]\UITests\RunTests.ps1' />
            </TestJob>
            ```

            At the end of your script you will need to copy the resulting trx file to the TestResults folder where Cloudtest expects to pick it up from. To generate a trx file, we used the mocha-trx-reporter npm package. To pass secrets to cloudtest, you can either use test secretstore which has been configured to use a certificate installed on all cloudtest VMs for particular paths, or one of the other solutions shown [here](https://stackoverflow.microsoft.com/questions/11589/getting-ais-token-in-cloudtest-machine/11665#11665)

            RunTests.ps1
            ```
            cd ..\UITests

            $env:USER_EMAIL = ..\StashClient\StashClient.exe -env:test pgetdecrypted -name:Your/Secret/Path -cstorename:My -cstorelocation:LocalMachine -cthumbprint:0000000000000000000000000000000000000000

            $env:USER_PASSWORD = ..\StashClient\StashClient.exe -env:test pgetdecrypted -name:Your/Secret/Path -cstorename:My -cstorelocation:LocalMachine -cthumbprint:0000000000000000000000000000000000000000

            $env:TEST_ENVIRONMENT = [environment]::GetEnvironmentVariable("TEST_ENVIRONMENT","Machine")

            mocha WorkspaceTests.js --reporter mocha-trx-reporter --reporter-file ./TestResults/result.trx

            xcopy TestResults\* ..\TestResults
            ```

            To pass non-secret parameters to your cloudtest session (and the msportalfx-tests) use the props switch when kicking off a cloudtest session. The properties will become machine level environment variables on your cloudtest VM. Once these are set as environment variables of the session, you can use nconf to pick them up in your UI test configuration.

            ```
            ct -t "amd64\CloudTest\TestMap.xml" -tenant Default -BuildId "GUID" -props worker:TEST_ENVIRONMENT=canary
            ```

    * Windows Azure Engineering System (WAES)

        See [WAES](http://aka.ms/WAES)

    * Jenkins


    *  How to setup test run parallelization