---
title: "How to Disable P4V Reviews in Perforce Swarm"
layout: post
---

With Perforce's deprecation of P4Web, you might be in the market for a new web tool for viewing code and diffs. If you don't want to fork up the cash for Fisheye, Perforce's new Swarm product will likely fit your needs.

I've got to hand it to Perforce, Swarm looks and works quite well, despite being implemented in PHP. It also comes with code review functionality that integrates directly with P4V.

However, if you are like my company, you may already have a code review tool that you want to use instead, and Swarm does not have an easy toggle for this. You've got to hack css and routes in order to disable it.

Perforce's original article on how to do this is [here](https://community.perforce.com/s/article/12679). Unfortunately, it completely leaves out how to disable the review option in P4V!

To completely disable reviews in P4V: in addition to following the steps in the above link, you also need to add the `'auto_registry_url' => false,` entry to Swarm's config.php in the `p4` config array. It should look something like the following after both changes have been made:

```php
<?php
return array(
    'environment' => array(
        'hostname' => 'myswarmhost.example.com',
    ),
    'p4' => array(
        'port' => 'perforce.example.com:1666',
        'user' => 'root',
        'password' => 'secret',
        'auto_register_url' => false,
    ),
    'router' => array(
        'routes' => array(
            'review'  => array('options' => array('route' => '')),
            'reviews' => array('options' => array('route' => '')),
            'home' => array(
                'options' => array(
                    'route'    => '[/]',
                    'defaults' => array(
                        'controller' => 'Changes\Controller\Index',
                        'action'     => 'changes',
                        'path'       => null
                    ),
                ),
            ),
        )
    ),
    'mail' => array(
        'transport' => array(
            'host' => 'mymailhost.example.com',
        ),
    ),
);
```

Once that's done, you can remove the Swarm URL from the Perforce server's configuration, which will prevent P4V from allowing reviews:

```
p4 property -d -n P4.Swarm.URL -s0
```

If for some reason the property comes back, double check your Swarm config.php again and make sure you have `auto_register_url` set to false in the `p4` config array.

I asked Perforce to update the article with this information, but it looks like they half-assed it and didn't provide any of the required specifics, just a vague sentence about disabling any auto registration features...