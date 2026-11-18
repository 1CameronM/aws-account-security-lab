# aws-account-security-lab
Secured a new AWS account with IAM, MFA, S3, CloudTrail, and GuardDuty, plus what went wrong and how I fixed it

# Secure an AWS Account (IAM, S3, CloudTrail, GuardDuty)

This is the first project on my cloud roadmap. I took a brand new AWS account and locked it down the way a real company would, then tested each piece to make sure it actually worked. Everything was done in the AWS console in my browser, in the Ohio region (us-east-2).

## What I built

* Locked down the root user with an authenticator app MFA, and made sure it has no access keys
* Made a separate admin user (`cameron-admin`) in an `admins` group, with its own MFA
* Set a $5 monthly budget alert so a mistake can't turn into a surprise bill
* Created an S3 bucket with public access blocked, versioning on, and default encryption
* Turned on a CloudTrail trail that covers all regions, with log file validation
* Turned on GuardDuty for threat detection

## Why I set it up this way

**IAM.** The root user can do anything in an account and can't be restricted, so I protected it with MFA and stopped using it for daily work. I log in as `cameron-admin` instead, so every action in the logs is tied to a named user and not root. I used a group so permissions live in one place. One honest note: `AdministratorAccess` is very broad. That's fine for a solo lab, but in a real company I'd scope it down to only what each person needs.

**S3.** Public buckets are one of the most common reasons companies leak data, so I turned on Block all public access first. Versioning lets me recover from accidental deletes or overwrites, and default encryption (SSE-S3) protects the files at rest.

**CloudTrail.** This is the audit trail. It records who did what, when, and from where. I turned on log file validation so I could prove the logs weren't tampered with. I skipped KMS encryption, data events, Insights, and CloudWatch Logs on purpose to avoid extra charges, and the logs are still encrypted by S3's default.

**GuardDuty.** This is the alarm system. It reads account activity (including CloudTrail) and raises findings when something looks suspicious.

## What I saw when I tested it

* CloudTrail Event history showed both of my `CreateBucket` actions under `cameron-admin`, not root
* GuardDuty's built in sample findings showed up as 479 entries labeled [SAMPLE], including a High severity EC2 Denial of Service finding. These are fake on purpose, so they just show what a real alert looks like
* Signing out and back in as `cameron-admin` asked for an MFA code

## Problems I ran into and how I fixed them

1. **My first AWS account was the wrong type.** It used AWS's new project based sign up, which has no root user and doesn't let IAM users log into the console. I made a second account with "Sign up for AWS (advanced)" so I could practice real root and IAM security.
2. **My admin user couldn't sign in.** It said "Authentication failed." I signed in as root and reset the console password for `cameron-admin`.
3. **The `admins` group had no permissions.** The Permissions column said "Not defined" because the policy didn't attach the first time. I attached `AdministratorAccess` from the group's Permissions tab. Lesson: always check the result after creating something.
4. **CloudTrail log file validation was off.** I'd selected it, but the trail details showed Disabled. I edited the trail and turned it on.
5. **GuardDuty wouldn't turn on.** The Free plan blocked it and sent me to an upgrade page. I upgraded to the Paid plan (my credits still get used first) and avoided AWS Organizations, since joining one wipes out the free credits.
6. **I enabled GuardDuty in the wrong region.** It went on in N. Virginia while everything else was in Ohio. I enabled it in Ohio and disabled the N. Virginia one so I wasn't paying twice.

## What I learned

* Root vs IAM users, and why MFA and least privilege matter
* Many AWS services are tied to a region, so check the region before you click anything
* A free plan can block services, and a budget alert warns you but doesn't stop spending
* Check every setting after you create it, because a few of mine didn't stick the first time

## Cost and cleanup

IAM and MFA are free, the first CloudTrail trail is free, and the S3 buckets cost almost nothing. GuardDuty has a 30 day free trial and then charges by usage, which would be tiny for an empty account. I'm turning it off before the trial ends.

## Screenshots

![IAM groups](screenshots/iam-groups.png)
![IAM users with MFA](screenshots/iam-users-mfa.png)
![S3 block public access](screenshots/s3-block-public-access.png)
![S3 versioning](screenshots/s3-versioning.png)
![CloudTrail trail details](screenshots/cloudtrail-trail.png)
![CloudTrail event history](screenshots/cloudtrail-event-history.png)
![GuardDuty sample findings](screenshots/guardduty-findings.png)

I blurred my account ID and IP address in all of these.
