# Adding/Removing Users Inject

#### Create & Transfer CSV

So in Libre office, you will have three coloumns, a `firstname`, `lastname`, and `username`. To do this, in libre office, have a formula for the third column, which will have the following value for the first column:
```
=LEFT(A1) &B1
```

`LEFT` takes the first character of the cell `A1`, `&` means both, and `B1` is the `lastname` cell. Now you will need to drag to of these together to make all of the cells. 

After that just type all of the first and last names, and then the usernames will be generated automatically in the third column. Then you can just save it as a CSV.

to copy the csv `users.csv` to the directory `/home/guyinatuxedo/` at the ip address `192.168.234.144` using hte user `guyinatuxedo` at `192.168.234.144`:
```
$	sudo scp /home/guyinatuxedo/Downloads/LocalSettings.php guyinatuxedo@192.168.234.144:/home/guyinatuxedo
```

#### Writing Scripts to Add users

First, it will be easier to escalate your privileges to root to do this:

```
$	sudo su
```

Next you are going to want to make the following script:

```
#	sudo vim make_users.sh
```

has the following code:

```
#!/bin/sh
echo -e "Enter password: " # Prompt and store the password in memory
read pw
#echo $pw
while IFS=, read -r col1 col2 col3 # Iterate through all rows 
do
	useradd $col3		# add the user
	echo -e "$col3:$pw" | chpasswd	# Assign the password to the user
done < users.csv	# Specify the file which we are reading from
```

and you can run it in the same directory as `users.csv`:
```
#	bash make_users.sh
```

also if you want a script to delete users from the csv `rem_users.sh`, here is the code for that:
```
while IFS=, read -r col1 col2 col3 # Iterate through all rows 
do
	userdel $col3  # Remove the user
done < rem_users.csv # Specify the file which we are reading from
```
