.. code:: robotframework

    *** Variables ***
    ${URL}       https://localhost/
    ${USERNAME}  st2admin
    ${PASSWORD}  st2admin

    *** Settings ***
    Library           Selenium2Library
    Library           Collections
    Library           Process
    Resource          ../keywords/WebDriver.rst
    Suite Setup       Connect Webdriver And Login  ${URL}  ${USERNAME}  ${PASSWORD}
    Suite Teardown    Close All Browsers

    *** Test Cases ***
    Test: Get "st2" and "bwc" version
        ${version}=  Run Process  st2  --version
        Log To Console  \n\nStackstorm version:-> ${version.stderr}

        # Temp fix for 2.5(Angular) and 2.6(react) UI
        ${ver}=  Set Variable If   "2.5" in "${version.stderr}"   2.5   2.6
        Set Global Variable    ${ST2_VERSION}    ${ver}
        Log To Console  \nST2 version to test UI: ${ST2_VERSION}\n
        #

        ${version}=  Run Process  bwc  --version
        Log To Console  \nBWC version:--------> ${version.stderr}\n

    Test: Verify Second tab is for Suites
        Set Selenium Implicit Wait    10 seconds
        Run Keyword If  ${ST2_VERSION}==2.6  Click Element  //*[@id="container"]/div/div/header/div[2]/a[2]
        ...       ELSE  Click Element  //html/body/div[1]/header/div[2]/a[2]

        Run Keyword If  ${ST2_VERSION}==2.6  Wait Until Element Is Visible  //*[@id="container"]/div/div/main/div[1]/div[3]/div  10  Seconds
        ...       ELSE  Wait Until Element Is Visible  //*[@id="st2-panel__scroller"]  10  Seconds

        ${result}  Get Location
        Should Contain  ${result}  suites
        Log To Console  \nSeccond Tab URL: ${result}\n

        # suites tab for v2.6 and above
        Run Keyword If  ${ST2_VERSION}==2.6  Wait Until Element Is Visible  //*[@id="container"]/div/div/main/div[1]/div[2]/div[1][contains(text(), 'Suites')]  10  Seconds
        ...       ELSE  Wait Until Element Is Visible  //html/body/div[1]/main/div[1]/div[2]/div[1][contains(text(), ' Suites ')]  10  Seconds


    Test: Verify that the Groupings of Actions/Workflows shows up in the DC Fabric Tab
        Run Keyword If  ${ST2_VERSION}==2.6  Wait Until Keyword Succeeds    3x  2s  Wait Until Element Is Visible   //*[@id="container"]/div/div/main/div[1]/div[3]/div/div[1]/div/h4/span[contains(text(), 'DCFABRIC')]    10  Seconds
        ...       ELSE  Wait Until Keyword Succeeds    3x  2s  Wait Until Element Is Visible   //html/body/div[1]/main/div[1]/div[3]/div/div[1]/div[1]/h4/span[contains(text(), ' DCFABRIC ')]    10  Seconds

        Log To Console    \nContains: "DCFABRIC"\n

        Run Keyword If  ${ST2_VERSION}==2.6  Wait Until Keyword Succeeds    3x  2s  Wait Until Element Contains  //*[@id="container"]/div/div/main/div[1]/div[3]/div/div[1]/div/h2  Manage EVPN Tenants and Edge Ports
        ...       ELSE  Wait Until Keyword Succeeds    3x  2s  Wait Until Element Contains  //*[@id="st2-panel__scroller"]/div[1]/div[1]/h2  Manage EVPN Tenants and Edge Ports
        Log To Console    \nContains: "Manage EVPN Tenants and Edge Ports" action\n
