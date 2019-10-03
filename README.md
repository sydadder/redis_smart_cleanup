# redis_smart_cleanup
To remove dormant/wrong redis entries on the server, I used the command line Shell Script to perform the cleanup. 

<h4>SOLUTION</h4>

````
redis-cli -h xxx.xx.xx.xxx -p 6380 keys "s:*:d:*" | 
while read LINE; 
do 
	IFS=':' read -r -a array <<< "$LINE"; 
	F360=redis-cli -h xxx.xx.xx.xxx -p 6380 hmget "$LINE" f360; 
		if [[ $F360 ]]; 
			then RESULT=mysql -u clientusernamepassword -pclientusernamepassword -h xxx.xx.xx.xxx flixcar -e "SELECT distributor_mpn_id FROM distributor_mpn where mpn = '${array[1]}' AND distributor_id = '${array[3]}';"; 
				if [[ -z $RESULT ]]; 
					then 
					echo "F360 Redis key exists. But no DB entry for $LINE : $F360 "; 
					echo "redis-cli -h xxx.xx.xx.xxx -p 6380 del \"$LINE\" f360" >> /home/syed/distributor_mpn_dormant_01AUG18.sh; 
				fi; 
		fi; 

		COMP=redis-cli -h xxx.xx.xx.xxx -p 6380 hmget "$LINE" comp; 
		if [[ $COMP ]]; 
			then 
			RESULT=mysql -u clientusernamepassword -pclientusernamepassword -h xxx.xx.xx.xxx flixcar -e "SELECT id FROM association_sku where sku = '${array[1]}' AND association_product_id = '$COMP' AND distributor_id = '${array[3]}';"; 
				if [[ -z $RESULT ]]; 
					then 
					echo "COMP Redis key exists. But no DB entry for $LINE : $COMP "; 
					echo "redis-cli -h xxx.xx.xx.xxx -p 6380 del \"$LINE\" comp" >> /home/syed/distributor_mpn_dormant_01AUG18.sh; 
				fi; 
		fi; 
done;

````
