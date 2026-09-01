# Lab 23: "Manhattan": can't write data into database.

## Description
 Your objective is to be able to insert a row in an existing Postgres database. The issue is not specific to Postgres and you don't need to know details about it (although it may help).

Helpful Postgres information: it's a service that listens to a port (:5432) and writes to disk in a data directory, the location of which is defined in the data_directory parameter of the configuration file /etc/postgresql/14/main/postgresql.conf. In our case Postgres is managed by systemd as a unit with name postgresql.

Test: (from default admin user) sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt

🔗 **Lab Link:** [SadServers - "Manhattan": can't write data into database.](https://sadservers.com/scenario/manhattan)

<br>

## 🪜 Steps

### Step 1: 

```bash
```
