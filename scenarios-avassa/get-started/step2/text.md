
Now lets view the sites.

```plain
./supctl show system sites
```{{exec}}

<br>

You can also get the summar of the status of the sites

```plain
./supctl show system site-status sites
```{{exec}}

<br>

In order to check a specific site enter tab after the command and select a specific site, for example
```plain
./supctl show system sites stockholm-sture
```{{exec}}

<br>

All of the above commands where targeted towards Control Tower. The Control Tower has a view of the state of the sites. But you can drill-down even further to sites by directing a supctl command to a specific site. For example, to get details abouts hosts on the site you can do:
```plain
./supctl show --site gothenburg-bergakungen system cluster hosts
```{{exec}}

<br>

The above command illustrates that on each site, the hosts form a cluster.
