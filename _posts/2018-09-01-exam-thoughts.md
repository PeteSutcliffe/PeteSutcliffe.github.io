---
published: true
title: Thoughts on the AWS Solutions Architect exam
---
I've taken a few Microsoft exams in my career and always been disappointed by them, particularly the programming focussed ones which have often contained glaring errors and ridiculous questions designed to trip you up rather than give you an opportunity to demonstrate what you know.

Happily this AWS exam was a big improvement on those. None of the questions felt unfair or poorly written and in most cases there was a fairly unambiguous right answer.

From reading about other people's experiences it sounds like there is quite a wide variety in the questions on offer. From what I recall my questions were spread across an assortment of topics:

- VPC design (public / private subnets, routing etc)
- Security groups / NACLS
- Lambda
- Dynamo DB
- Cloudfront
- Elastic Container Service
- Elasticache
- Redshift
- EBS (what different types are suitable for)
- SNS / SQS
- Cross regional backups (S3, Red Shift, EBS etc)

Most of the questions were scenario-based and, outside of the VPC focussed questions which required more detailed understanding, mostly just required choosing the right technology for the scenario presented. High volume of database queries causing issues? Stick Elasticache in there, need a data warehouse? Use Redshift, that kind of thing.

To prepare for the exam I watched all the videos on [A Cloud Guru](https://acloud.guru), the content is good and covers all the major exam areas but is probably not quite detailed enough to cover everything that might come up. Their final practice exam is also terrible, with questions focussing on low-level technical details that are highly unlikely to come up in the exam (certainly I didn't get any like that). I read the AWS FAQs for most of the big services, which contain some useful details. I read through the AWS well-architected whitepapers as well but these were pretty dull and I don't feel like they helped with the exam at all.
I used some sample questions from Whizlabs, some of the questions were quite-poorly written and there was a lot of repetition but they have quite detailed explanations of why answers are wrong or right and links to relevant AWS documentation, which is very helpful. There are some exam preparation courses on Udemy as well, with free sample questions which are worth checking out.
