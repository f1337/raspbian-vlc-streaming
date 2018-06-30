# Hardware-encoded H.264 video streaming using VLC on Raspberry Pi

## Caveats:

I have tested the following on Raspbian 8 (jessie) using a Logitech C920 webcam. YMMV with other distros, versions and cameras. Kudos and  :beers: are owed to [the VLC team](https://wiki.videolan.org/Documentation:Streaming_HowTo/Advanced_Streaming_Using_the_Command_Line/#access), [Matthias Bock](https://wiki.matthiasbock.net/index.php/Logitech_C920,_streaming_H.264) and [Lars K.W. Gohlke](https://blog.lgohlke.de/2014/02/09/babyphone-mit-dem-raspberry-pi.html).

## Streaming Video via HTTP

1. Upgrade the Raspberry Pi firmware:

        sudo apt-get update
        sudo apt-get install rpi-update raspi-config
        sudo rpi-update
        sudo reboot

1. Test HTTP streaming manually:

        cvlc v4l2:///dev/video0:chroma=h264:width=800:height=600 :input-slave="alsa://hw:1,0" --sout '#transcode{aenc=ffmpeg{strict=-2},acodec=mp4a,ab=32}:http{mux=ts,dst=:8080}' -vvv
    *Notes*:

      * Using the `mp4a` audio codec will allow you to stream from your Mac, PC and iOS devices.
      * Raspbian 7 (wheezy) did not require `aenc=ffmpeg{strict=-2},` in the `#transcode` config.

1. In a [VLC client](https://www.videolan.org/) on your computer or phone, open http://192.168.0.x:8080/. If the audio and video from the camera are not streaming, something went wrong.

1. Copy the init script to your pi:

        scp etc/init.d/vlc pi@192.168.0.x:initd-vlc

1. Install init script & start automatically

        ssh pi@192.168.0.x
        sudo mv ~/initd-vlc /etc/init.d/vlc
        sudo chmod 755 /etc/init.d/vlc
        sudo update-rc.d vlc defaults
        sudo systemctl daemon-reload
        sudo service vlc start

1. Re-open http://192.168.0.x:8080/ in the VLC client. You should now see/hear streaming video/audio.

## Add an HTTPS Proxy

1. Use [Certbot](https://certbot.eff.org/) to get a free SSL certificate from Let's Encrypt. I used the following:

        sudo certbot certonly --standalone --agree-tos -n -m example@example.com -d example.com

1. Install nginx:

        sudo apt-get install nginx

1. Create user(s):

        sudo htpasswd -c /etc/nginx/.htpasswd user1

1. Change all mentions of `example.com` with your domain in `etc/nginx/sites-available/vlc`, then copy the nginx config to your pi:

        scp etc/nginx/vlc pi@192.168.0.x:nginx-vlc

1. Install nginx config & start:

        ssh pi@192.168.0.x
        sudo mv ~/nginx-vlc /etc/nginx/sites-available/vlc
        sudo systemctl start nginx

## Setup Certbot Auto-renewal

1. `crontab -e`, and add the following:

        43 0 * * * sudo certbot renew --quiet --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx"

## TODO

* update instructions for `systemd`?
