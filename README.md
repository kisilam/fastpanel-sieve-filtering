# fastpanel-sieve-filtering
How to enable sieve filters in the Fastpanel

1. Need install packages (it`s for Ubuntu, Debian)
   *  dovecot-sieve
   *  dovecot-managesieved
   *  dovecot-lmtpd
  
2. Іn /etc/dovecot/10-mail.conf add this settings

                  #manual editing
                  service stats {
                    unix_listener stats-reader {
                      group = 
                      mode = 0666
                    }
                    unix_listener stats-writer {
                      group = 
                      mode = 0666
                    }
                  }

 3. Іn /etc/dovecot/15-lda.conf use this settings

          # Address to use when sending rejection mails.
          # Default is postmaster@<your domain>.
          postmaster_address = postmaster@%d

          --------

          protocol lda {
              # Space separated list of plugins to load (default is global mail_plugins).
              mail_plugins = $mail_plugins sieve
              }

  4. In /etc/dovecot/20-lmtp.conf  use this settings

          protocol lmtp {
              # Space separated list of plugins to load (default is global mail_plugins).
              mail_plugins = $mail_plugins sieve
             }
  
  5. I add 20-managesieve.conf and 90-sieve.conf in repo. You can use it or use only some settings from them.

  6. Edit /usr/share/fastpanel2-roundcube/config/config.inc.php for adding the megasieve plugin to the Roundcube

        $config['plugins'] = [
                      \'archive\',
                      \'zipdownload\',
                      \'managesieve\',
                        ];
  
  7. Add this settings to the /etc/exim4/exim4.conf.template in #the transports configuration

         # LDA dovecot
            dovecot:
          	driver = pipe
          	command = /usr/lib/dovecot/dovecot-lda -e -d $local_part@$domain -f $sender_address -a $original_local_part@$original_domain
          	return_path_add
          	log_output = true
          	delivery_date_add
          	envelope_to_add
          	user = ${extract{1}{:}{${lookup{$local_part@$domain}lsearch{EXIM_PASSWD}}}}
          	group = ${extract{2}{:}{${lookup{$local_part@$domain}lsearch{EXIM_PASSWD}}}}
          	return_output
    
    and edit this line in #routers definition

        local_users:
            driver = accept
            #transport = local_maildir
            transport = dovecot
            condition = ${lookup {$local_part@$domain} lsearch {EXIM_PASSWD} {yes} {no} }

That's all.
Don't forget to restart exim4 and dovecot service

    service exim4 restart; service dovecot restart
  
  

