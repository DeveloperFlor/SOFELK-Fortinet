# Adding Fortinet support to SOF-ELK

---

`Title` Adding Fortinet support to SOF-ELK

`Author` Flor Hemmeryckx

`Date` 03/05/2023

---

## Installing

1. Download and install the SOF-ELK VM
   [Documentation](https://github.com/philhagen/sof-elk/blob/main/VM_README.md)
   [Download](https://for572.com/sof-elk-vm)

2. Copy the ansible configuration files to the VM
   
   ```bash
   scp .\SOFELK-Fortinet\* elk_user@{VM_IP}:/home/elk_user/
   forensics
   ```

3. Run the ansible playbook
   
   ```bash
   ssh elk_user@{VM_IP}
   forensics
   sudo ansible-playbook -i hosts.ini ansible-fortinet.yaml -b
   forensics
   ```

4. Add your fortinet logfiles to the logstash folder: `/logstash/fortinet/`

5. Wait for logstash to process the new logfiles
   
   http://{VM_IP}:5601/app/management/data/index_management/indices

6. A basic dashboard is already created an has its data loaded (if you waited long enough ;-))

---

## Technical notes

### `fortinet.conf`

Contains the logstash configuration file that:

- Read log files from `/logstash/fortinet/`

- Depending on if its a .csv or .log uses another way to process the keyvalue's

- Makes sure the epoch time format is alway 13 chars long

- Binds the epoch time to `@timestamp`

- Converts:
  
  - `rcvdbyte` => `integer`
  
  - `rcvdpkt` => `integer`
  
  - `sentbyte` => `integer`
  
  - `sentpkt` => `integer`

- Sends all this to an index `fortinet-logs-%{+YYYY.MM.dd}`

### `ansible-fortinet.yaml`

Contains the ansible playbook configuration that:

- Creates the logstash config: _converts the logs to indices_

- Creates the fortinet directory at: `/logstash/fortinet/`

- Creates a sincedb file for the logstash indices at: `/var/log/logstash/sincedb`

- Creates the index-patern: _makes it possible to load the logs in discover/dashboard)_

- Creates the index-template: _converts strings to ip's_

- Restarts the logstash service after its done

## `hosts.ini`

Just a file to specify the hosts the ansible script has to run on (localhost)



**Important: If for some reason you want to re-run the ansible playbook. Make sure that you remove the previously created index-template & data view. otherwise you will end up with duplicates (it will still work, but can be confusing)**
