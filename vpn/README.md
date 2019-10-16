# OpenVPN setup

## IAM perms

```
aws iam attach-role-policy --role-name bastion-role --policy-arn arn:aws:iam::accountid:policy/openvpn-access-s3-rw-access
```

## OpenVPN Client config setup
```
aws s3 cp openvpn-client.conf s3://openvpn-access-region-accountid/config/openvpn-client.conf
```

## Allow openvpn traffic to bastion
```
aws ec2 authorize-security-group-ingress --group-id GroupId --protocol udp --port 1194 --cidr YourIp
```

## docker install
```
sudo -s
apt-get update
apt-get install docker.io -y
systemctl enable docker
systemctl start docker
docker run -v /etc/openvpn:/etc/openvpn --log-driver=none --rm kylemanna/openvpn ovpn_genconfig -u udp://$(hostname -f)
```

* Fill out the vars file and copy paste into /etc/openvpn/vars, then:

```
docker run -v /etc/openvpn:/etc/openvpn --log-driver=none --rm kylemanna/openvpn bash -c "cd /etc/openvpn && ovpn_initpki nopass"
docker run -v /etc/openvpn:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN --name openvpn kylemanna/openvpn
```

## Routing (optional)
```
echo 'push "route 10.0.0.0 255.255.0.0"' >> /etc/openvpn/openvpn.conf
docker restart openvpn
```

## sync config
```
sudo -s # if you're not root yet
apt install awscli
aws s3 sync /etc/openvpn s3://openvpn-access-region-accountid/config/
```

## VPN Clients
* https://openvpn.net/client-connect-vpn-for-windows/ (MacOS + Windows)
* https://tunnelblick.net/downloads.html (recommended for MacOS)
