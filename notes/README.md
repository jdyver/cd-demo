## Add Jenkins Build and Publish
Do that ^^
- Only required for the later versions of Jenkins (3.5.4-2.150.1+)

### Here are the detailed screenshot steps
![Jenkins - UI](https://github.com/jdyver/cd-demo-jd/blob/master/notes/img/1JenkinsUI.png)
- Manage Jenkins > 
![Jenkins - ManageJenkins](https://github.com/jdyver/cd-demo-jd/blob/master/notes/img/2ManageJk.png)
- [Scroll Down] Manage Plugins > 
![Jenkins - ManagePlugins](https://github.com/jdyver/cd-demo-jd/blob/master/notes/img/3ManagePl.png)
- [Select] Pipeline: Step API > 
![Jenkins - SelectPlugin](https://github.com/jdyver/cd-demo-jd/blob/master/notes/img/4aSelectPl.png)
- Select [Download now and install after restart] > 
![Jenkins - SelectDownload](https://github.com/jdyver/cd-demo-jd/blob/master/notes/img/4bDownloadInstall.png)
- Select [Restart Jenkins when installation is complete and no jobs are running]
![Jenkins - Restart](https://github.com/jdyver/cd-demo-jd/blob/master/notes/img/5Restart.png)
- Refresh page after a few minutes if it doesn't do it automatically

### Check for option in new job > Build > [Add build step]
![Jenkins - Validated](https://github.com/jdyver/cd-demo-jd/blob/master/notes/img/6Validate.png)