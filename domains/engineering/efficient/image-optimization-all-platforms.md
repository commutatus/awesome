---
title: Run Image Optimization On All Platforms
nav_order: 7
parent: Efficient
grand_parent: Engineering
---

# Run image optimization on all locally stored assets on build

When shipping code and hitting deadlines, it’s easy to forget about optimizing images. Web pages with optimized images load faster. Faster pages have higher conversion rates, lower bounce rates, and happier users. And of course, smaller images reduce bandwidth costs for us and our visitors.

1. ToC
{:toc}

### How it works

We're currently using [Imgbot](https://imgbot.net/) to optimize our images and also automate the process of Image Optimization. Once set up, Imgbot regularly optimizes images from our projects and creates a Pull Request to be merged with the master branch for deployment.

### How to set up 

Imgbot needs to be set up by the Admin of Organisation account in Github or by the Owner of any personal project since it requires purchasing plans. Open source set up however is free for all. 

The set up is straightforward with selecting the required plan and purchase it.
Then, choose if you would like to give access to Imgbot to optimize images for all of your projects or any select project and complete the setup process.

### How to manage Imgbot Optimization

Most of the time, it doesn't require to be managed as it automatically optimizes images and creates a PR. In cases where you need to configure how the optimization should function, you can follow their [docs](https://imgbot.net/docs/).


### References

[https://imgbot.net/](https://imgbot.net/)

[https://imgbot.net/docs/](https://imgbot.net/docs/)
