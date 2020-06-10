# ATLAS Catalogues
Catalogues form a collection of one or more security controls.  By default, ATLAS does not ship with any catalogues loaded.  Upon initial installation of ATLAS, you will need to load the applicable catalogues for your organization to make use of them throughout the application.

## Steps to Load Catalogues
1.  Review the list of available catalogues within ATLAS:
    - [California Consumer Privacy Act (CCPA)](ccpa.json)
    - [Cybersecurity Maturity Model Certification (CMMC) 1.0](cmmc.json)
    - [Cloud Security Alliance (CSA) Cloud Controls Matrix (CCM) 3.0.1](csa-ccm-301.json)
    - [Financial Services Sector Cybersecurity Profile (FSSCP) v1.0](fssc.json)
    - [General Data Protection Regulation (GDPR)](gdpr.json)
    - [NIST SP800-53 Rev 4](nist800-53-r4.json)
    - [NIST Cybersecurity Framework (CSF) 1.1](nist-csf-v1-1.json)
2.  Log into ATLAS with an Admin account (NOTE: Can be any user account with the Administrator role applied)
3.  Click your username in the top right and select Catalogues 
4.  Click the 'Create New' button
5.  Click the 'Import/Export' button 
6.  Choose the appropriate JSON file from above to upload
7.  Click the 'Import Controls' button
8.  Catalogue is now fully loaded with the controls
9.  Repeat steps above for each catalogue your organization would like loaded

## Using Catalogues
1.  Create a Security Profile by selecting one or more controls in one or more catalogues.  Profiles become templates to create/instantiate security plans.
2.  Once profiles are created, create a new security plan.
3.  Use the Security Plan Builder to create a templated security plan based on the profile and manual controls selected.
4.  After you finish the builder, a new plan will be created using the security controls in the profile.
5.  You can create as many security profiles and security plans as needed.
