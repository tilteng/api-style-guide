When you run an integration test, it creates a temp db to store objects created by the integration test.   
For example running any integration test, such as `sudo prove -lrv t/01-users/01-basic.t` would result in one of the couple first lines to say:
```
NOTICE: TEST t/01-users/01-basic.t IS USING TEST DATABASE crowdtilt_testing_root_1482033253_6313
```
If your test fails and you have no idea why and you want to see exactly what objects were created with what info, you can do:
```
sudo -u postgres psql  crowdtilt_testing_root_1481784166_10956
```
and play around in the test db that your integration test used. 


