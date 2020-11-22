# public-coredns-tls

This project builds on the blog below to enable a self hosted DNS over TLS solution with PiHole filtering that can be run from a cloud provider and have fully automated certificates from a public CA.
Original author of much of this content:
https://bartonbytes.com/posts/how-to-configure-coredns-for-dns-over-tls/

Pre-Reqs:
- 2 seperate fully configured docker hosts with docker-compose -- This is required as of now since I have not found a good way to allow the pihole container and coredns container to play well together on the same host since they both use port 53.
- You will most likely need to stop & disable your the DNS service on both Docker hosts. Steps for CentOS 8:
   ```
   sudo systemctl stop systemd-resolved.service
   sudo systemctl disable systemd-resolved.service
   ```
- Pihole fully configured and running on one of the docker hosts.
- Public DNS entry that resolves to the public ip address of the docker host that will run coredns for DNS over TLS.

Steps to setup:
1. Clone this repository onto the docker host NOT running pihole. My examples assume you have cloned it into your users home directory.
2. Update the DNSHOSTNAME variable in the configure.sh and renew.sh files to contain the valid public dns name that resolves to your docker host.
3. Setup your docker-compose.yml and coreconfig-up/down files as described by Grants blog (https://bartonbytes.com/posts/how-to-configure-coredns-for-dns-over-tls/). Dont worry about his certificate steps.
   - Note: The certificate paths listed in the provided coreconfig-down file do not need to be updated as long as you use the default letsencrypt directories in the configure.sh and renew.sh scripts.
4. Run ./configure.sh
5. At this point, you should have a functioning self hosted DNS over TLS service. You can test with with some of the following commands:
   - Validate your certificate:
   > openssl s_client -connect \<corednsdockerhostpublicDNSHost\>:853 -servername \<corednsdockerhostpublicDNSHost\>
   - Test full DNS functionality over TLS:
   > kdig -d @\<corednsdockerhostpublicIP\> +tls-ca +tls-host=\<corednsdockerhostpublicDNSHost\> amazon.com
5. Now, if you're familiar with letsencrypt certificates, you will know that they expire every 90 days. Let's address this by setting up some automation to renew the cert. To do this we will simply setup a cronjob as seen below that will run the certbot container to renew the certificate previously configred by the configure.sh script.
   > 0 2 */10 * * /home/\<yourusername\>/public-coredns-tls/renew.sh &> ~/letsencrypt_renew.log
   - Note: This job will run every 10 days and by default if the certbot detects no certs are due for renewall it will leave the cert as is.
