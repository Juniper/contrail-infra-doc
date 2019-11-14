SSL certs
=========

review.opencontrail.org
-----------------------

Certificate for https://review.opencontrail.org was issued by Let's Encrypt CA with use of certbot.

It was something like `set&forget`, because cert is auto-renewed by certbot run periodically by cron.

The configuration was done following this instruction:

.. code:: bash

  # Installing certbot
  wget https://dl.eff.org/certbot-auto
  sudo mv certbot-auto /usr/local/bin/certbot-auto
  sudo chown root /usr/local/bin/certbot-auto
  sudo chmod 0755 /usr/local/bin/certbot-auto

  # Issuing the cert and configuring Apache
  sudo /usr/local/bin/certbot-auto --apache

  # Setting crontab for auto-renewal
  echo "0 0,12 * * * root python -c 'import random; import time; time.sleep(random.random() * 3600)' && /usr/local/bin/certbot-auto renew" | sudo tee -a /etc/crontab > /dev/null

(source: https://certbot.eff.org/lets-encrypt/ubuntuother-apache)
