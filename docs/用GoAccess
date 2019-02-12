# 用GoAccess实现可视化并实时监控access日志

Get Started
Here are the basic steps on how to get started with GoAccess v1.3
1. Download GoAccess
First and foremost you need to download GoAccess. There are several ways to download GoAccess. Choose one of the following options:

Download and build from source (tar.gz)
Use your preferred package manager of your Linux distribution.
Build from development and get the latest and greatest.
Build GoAccess' Docker image from upstream.
2. Determine Log Format
Once you have GoAccess installed on a machine, then you should be ready to start using it. However, first you need to determine the log format of your access log. GoAccess comes with several predefined log format options that you can use. Either you can set them permanently in your configuration file or simply passing it through the command line.

If you are unsure, please feel free to open a new issue on Github and post a few sample lines from your access log. If you have a custom log format, please take a look at the custom log format options.

3. Run GoAccess
At this point you are ready to run GoAccess against your access log(s). The following are the most basic and common scenarios.

3.1 Terminal Output
The following prompts a log configuration dialog with predefined log formats for you to choose from and then displays the stats in real-time.

 goaccess access.log -c
3.2 Static HTML Output
The following parses the access log and displays the stats in a static HTML report.

 goaccess access.log -o report.html --log-format=COMBINED
 Note
Here we specify the log format directly in the command line using --log-format. You can also specify the log format in your configuration file as described in the actual config file.
3.3 Real-Time HTML Output
The following parses the access log and displays the stats in a real-time HTML report.

 goaccess access.log -o /var/www/html/report.html --log-format=COMBINED --real-time-html
 Important
You should place your report.html output file under your Web Server document root.
You should be able to simply open your report.html by navigating the browser to your document root URL. e.g., http://example.com/report.html
GoAccess features its own Web Socket server and that's how it pushes the latest data to the browser.
If you don't run a Web Server to host your report.html, you can simply open the output file through your browser (Ctrl+o).
Troubleshooting
Take a look at the Man Page for details, examples and some neat stuff you can do with GoAccess.
Check the FAQ.
Still not sure or found a bug? Please open a new issue on Github.
