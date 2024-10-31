## Targets
**The all targer**

Make multiple targets and you want all of them to run? Make an all target. Since this is the first rule listed, it will run by default if make is called without specifying a target.

![alt text](image-13.png)

Result:

![alt text](image-14.png)

**Multiple targets**

When there are multiple targets for a rule, the commands will be run for each target. $@ is an automatic variable that contains the target name.

![alt text](image-15.png)

Result:

![alt text](image-16.png)
