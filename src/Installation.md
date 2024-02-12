# Installing & seting up FPGA 


## Installing Vivado

1. If you dont have a Xilinx account, create a free account, using url below:
   - [login to Xilinx](https://login.amd.com/app/amd_accountamdcom_1/exk559qg7f4aW4yim697/sso/saml?SAMLRequest=fVLRTsIwFP2Vpe%2Bsc2ywNYxkQowkqARQE19I6Tpo7NrR2yn8vWWg4oM89eb23HNuz%2BkAaCVrkjd2q%2BZ813Cw3r6SCkh7kaHGKKIpCCCKVhyIZWSRP0xJ6AekNtpqpiXycgBurNBqpBU0FTcLbj4E48%2FzaYa21tZAMKaM6UZZn1aFz3SFjwoYauxoSiE5rjVYBwLkjd0aQtEj4e%2B41BuhfoZpXWNXr86krnTd1Q3m%2B%2Fc4TnebfhnR1%2Bggql7axwC6VUPeZJyhVVwmtEzLtBf3aa9MWDdas7DXLfpxkhRJUTBWsCh1YICGTxRYqmyGwiCMOkHYCdJlkJIwIHHiB0H4hrw7bRhvLcxQSSVw5M3O1twKVQi1ue7j%2BgQCcr9czjqzp8USeS%2FcQPt8B0DDwXF70u5jLvK5Tku%2FQ0HD%2FyJw57k1wBcSJ72aPDrOyXimpWAHL5dSf44Mp5ZnyJqGIzw8Tf39P8Mv&RelayState=https%3A%2F%2Faccount%2Eamd%2Ecom%2Fen%2Fprofile%2Ehtml)
2. Download the AMD Unified Installer for FPGAs & Adaptive SoCs 2023.2: Linux Self Extracting Web Installer from below link:
   - [vivado](https://www.xilinx.com/support/download.html)

3. Make the Vivado installer executable and run it using:
   - ``` chmod +x Xilinx_*.bin ```
   - ``` sudo ./Xilinx_*.bin ```
4. Once the installer loads2
  - ``` click "Next" ```
5. Now enter your Xilinx username and password. Then Click "Next".
6. Agree to all three statements and Click "Next". Incase, you disagree you can’t
proceed further.
7. Select "Vivado HL WebPACK or Vivado Lab " and click "Next".
8. Click "Reset to Defaults" and then press "Next". 3
9. By default, the "installation directory" is "/tools/Xilinx". This is the default installation
directory. Click "Next".
10. Click "Install" and wait for the installer to finish.
11. Install the Xilinx cable drivers:
   - ``` cd /tools/Xilinx/Vivado/2018.3/data/xicom/cable_drivers/lin64/install_script/install_drivers ```
   - ``` sudo ./install_drivers ```
12. Do some permissions cleanup:
    - ``` cd ```
    - ``` cd .Xilinx/Vivado ```
    - ``` sudo chown -R $USER * ```
    - ``` sudo chmod -R 777 * ```
    - ``` sudo chgrp -R $USER * ```
13. Add Vivado path to the environmental variable PATH in .bashrc :
    - ``` export PATH=$PATH:/tools/Xilinx/Vivado/2018.3/bin ```
    - ``` export PATH=$PATH:/tools/Xilinx/SDK/2018.3/bin   ```
14. Test Vivado
   - ``` vivado -version ```

## Programming Arty-7 FPGA

Refer [shakti user manual](https://shakti.org.in/docs/shakti-soc-user-manual.pdf) to program the board.
