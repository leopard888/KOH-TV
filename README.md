# KOH-TV
IPTV  KOH
How To Install Nginx RTMP Server + HLS on Ubuntu 16.04 

Install the tools required to compile Nginx and Nginx-RTMP from source.
------------------------------------------------------------------------

sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev


Download the Nginx and Nginx-RTMP source.
--------------------------------------------------------------

wget http://nginx.org/download/nginx-1.14.0.tar.gz

wget https://github.com/arut/nginx-rtmp-module/archive/master.zip


Install the Unzip package.
--------------------------------------------

sudo apt-get install unzip


Extract the Nginx and Nginx-RTMP source.
------------------------------------------------

tar -zxvf nginx-1.14.0.tar.gz
unzip master.zip


Switch to the Nginx directory.
-------------------------------------------------

cd nginx-1.14.0


Add modules that Nginx will be compiled with. Nginx-RTMP is included.
--------------------------------------------------------------------------

./configure --with-http_ssl_module --add-module=../nginx-rtmp-module-master


Compile and install Nginx with Nginx-RTMP.
-------------------------------------------------------

make
sudo make install


Now Install the Nginx init scripts.
-----------------------------------------------------------------
sudo wget https://raw.github.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx

sudo chmod +x /etc/init.d/nginx
sudo update-rc.d nginx defaults


Update the package lists.
-----------------------------------------------------------------
sudo apt-get update


Install FFmpeg.
--------------------------------------------------------------------
sudo apt-get install ffmpeg


Create the folder structures necessary to hold the live and mobile HLS manifests and video fragments:
---------------------------------------------------------------------
sudo mkdir /HLS
sudo mkdir /HLS/live
sudo mkdir /HLS/mobile
sudo mkdir /video_recordings
sudo chmod -R 777 /video_recordings


It’s probably a good idea to have your firewall turned on if you haven’t done so already. If so, you must allow traffic into the ports used by Nginx and HLS. If you’d like to run without the firewall for now, ignore the ufw section below.
-----------------------------------------------------------------------------
sudo ufw limit ssh
sudo ufw allow 80
sudo ufw allow 1935
sudo ufw enable


Now open the nginx configuration file
------------------------------------------------------------------------------
sudo nano /usr/local/nginx/conf/nginx.conf


ลบโค๊ดทั้งหมดแล้วเอาไฟล์ด้านล่างไปใส่แทน
---------------------------------------------------------

#user nobody;
worker_processes  1;

error_log  logs/rtmp_error.log debug;
pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    server {
        listen       80;
        server_name  localhost;

        location /hls {
            # Serve HLS fragments

            # CORS setup
            add_header 'Access-Control-Allow-Origin' '*' always;
            add_header 'Access-Control-Expose-Headers' 'Content-Length';

            # allow CORS preflight requests
            if ($request_method = 'OPTIONS') {
                add_header 'Access-Control-Allow-Origin' '*';
                add_header 'Access-Control-Max-Age' 1728000;
                add_header 'Content-Type' 'text/plain charset=UTF-8';
                add_header 'Content-Length' 0;
                return 204;
            }


            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /tmp;
            add_header Cache-Control no-cache;
        }
    }
}

rtmp {
        server {
                listen 1935;
                chunk_size 8192;

                application hls {
                        live on;
                        meta copy;
                        hls on;
                        hls_path /tmp/hls;
        }
    }
}



Ctrl + X to save the file to disk and exit.

Before you do anything else, it’s important to take care of what is called “cross-domain” restrictions, which would otherwise shut down your ability to stream to a webpage/website. Create a crossdomain.xml file in your nginx/html folder and put instructions in it to allow data to flow between domains:
----------------------------------------------------------------
sudo nano /usr/local/nginx/html/crossdomain.xml


First copy (from this page) and then paste (right-click) into the nano editor field the following XML data:
------------------------------------------------------------------
<?xml version="1.0"?>
<!DOCTYPE cross-domain-policy SYSTEM "http://www.adobe.com/xml/dtds/cross-domain-policy.dtd">
<cross-domain-policy>
<allow-access-from domain="*"/>
</cross-domain-policy>


Press Ctrl + O to write out, then Ctrl + X to save the file to disk and exit.
And your done!

little info about your servers
FMS URL: rtmp://your-ip/hls/

Stream Key: ANY thing you like, I usually set it to stream.
http://your-ip/hls/stream.m3u8

again if you set your stream key to movie then it would look something like this
http://your-ip/hls/movie.m3u8
