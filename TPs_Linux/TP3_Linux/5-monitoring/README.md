# Module 5 : Monitoring

Dans ce sujet on va installer un outil plutôt clé en main pour mettre en place un monitoring simple de nos machines.

L'outil qu'on va utiliser est [Netdata](https://learn.netdata.cloud/docs/agent/packaging/installer/methods/kickstart).

➜ **Je vous laisse suivre la doc pour le mettre en place** [ou ce genre de lien](https://wiki.crowncloud.net/?How_to_Install_Netdata_on_Rocky_Linux_9). Vous n'avez pas besoin d'utiliser le "Netdata Cloud" machin truc. Faites simplement une install locale.

Installez-le sur `web.tp2.linux` et `db.tp2.linux`.

```
[audran@web ~]$ sudo dnf install epel-release -y
Complete!

[audran@web ~]$ sudo wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh
Successfully installed the Netdata Agent.

[audran@web ~]$ sudo systemctl start netdata
[audran@web ~]$ sudo systemctl enable netdata
```

Une fois en place, Netdata déploie une interface un Web pour avoir moult stats en temps réel, utilisez une commande `ss` pour repérer sur quel port il tourne.

```
[audran@web ~]$ sudo ss -alnpt
LISTEN         0               4096                           0.0.0.0:19999                        0.0.0.0:*
users:(("netdata",pid=4800,fd=6))

[audran@web ~]$ sudo firewall-cmd --add-port=19999/tcp --permanent
[audran@web ~]$ sudo firewall-cmd --reload
```

Utilisez votre navigateur pour visiter l'interface web de Netdata `http://<IP_VM>:<PORT_NETDATA>`.

Après les mêmes manipulations sur les deux machines, on a accès à l'interface web de Netdata du serveur web et de la base de donnée.

```
http://10.102.1.11:19999/#after=-540;before=0;;theme=slate;utc=Europe%2FParis

http://10.102.1.12:19999/#after=-540;before=0;;theme=slate;utc=Europe%2FParis
```

➜ **Configurer Netdata pour qu'il vous envoie des alertes** dans [un salon Discord](https://learn.netdata.cloud/docs/agent/health/notifications/discord) dédié en cas de soucis

```
[audran@web ~]$ sudo nano /etc/netdata/health_alarm_notify.conf
###############################################################################
# sending discord notifications

# note: multiple recipients can be given like this:
#                  "CHANNEL1 CHANNEL2 ..."

# enable/disable sending discord notifications
SEND_DISCORD="YES"

# Create a webhook by following the official documentation -
# https://support.discordapp.com/hc/en-us/articles/228383668-Intro-to-Webhooks
DISCORD_WEBHOOK_URL="https://discordapp.com/api/webhooks/1044058582234181714/sGCbBWmAugHk3d98p9OM_xlwktNtLfTX3Lf_O-1Zmm>

# if a role's recipients are not configured, a notification will be send to
# this discord channel (empty = do not send a notification for unconfigured
# roles):
DEFAULT_RECIPIENT_DISCORD="général"
```

```
[audran@web ~]$ sudo nano /etc/netdata/health.d/cpu_usage.conf
alarm: cpu_usage
on: system.cpu
lookup: average -3s percentage foreach user, system
units: %
every: 10s
warn: $this > 50
crit: $this > 80
info: CPU utilization of users on the system itself.
```

```
[audran@web ~]$ sudo systemctl restart netdata
```

➜ **Vérifier que les alertes fonctionnent** en surchargeant volontairement la machine par exemple (effectuez des *stress tests* de RAM et CPU, ou remplissez le disque volontairement par exemple)

![Monitoring](../pics/monit.jpg)