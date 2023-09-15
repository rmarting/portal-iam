minikube delete
minikube start
minikube addons enable ingress
helm dependency update
helm install centralidp -f ../../consortia/environments/centralidp/values-local.yaml .
helm upgrade centralidp -f ../../consortia/environments/centralidp/values-local.yaml .

minikube ip
kubectl get ing
minikube service list
kubectl get pods -n ingress-nginx
adjust hosts file

Dump:
psql -h centralidp-postgresql-primary -d iamcentralidp -U postgres -W


https://github.com/bitnami/charts/issues/8025
https://github.com/bitnami/charts/issues/14926


    command:
    - /bin/sh
    - '-c'
    args:
    - sleep infinity

unset POSTGRES_DATABASE

# lets work in /tmp
cd /tmp/

# Download binaries of your old installation.
# I found this line as quite helpful to identify the url
# https://github.com/bitnami/containers/blob/2ab7f4b1fc3ae7f8c0c18a38e4ec078ccd0c2d89/bitnami/postgresql/15/deb

# Download binaries of your old installation.
# I found this line as quite helpful to identify the url
# https://github.com/bitnami/containers/blob/2ab7f4b1fc3ae7f8c0c18a38e4ec078ccd0c2d89/bitnami/postgresqlian-11/Dockerfile#L35
# Following line for debian 11 with postgresql 11
# hat nicht funktioniert:
perl -e 'use IO::Socket::INET; my $s = IO::Socket::INET->new(PeerAddr=>"downloads.bitnami.com", PeerPort=>80, Proto=>"tcp"); print $s "GET /files/stacksmith/postgresql-14.2.0-0-linux-amd64-debian-11.tar.gz HTTP/1.0\r\nHost: downloads.bitnami.com\r\n\r\n"; while(<$s>) { last if $_ eq "\r\n" }; open(my $f, "$FILE_NAME"); print $f $_ while <$s>; close $f'
tar --extract --directory /bitnami/postgresql/oldbin/ --file postgresql-14.2.0-0-linux-amd64-debian-11.tar.gz --strip-components=4 postgresql-14.2.0-0-linux-amd64-debian-11/files/postgresql/bin

## Works
perl -e 'use IO::Socket::INET; my $s = IO::Socket::INET->new(PeerAddr=>"downloads.bitnami.com", PeerPort=>80, Proto=>"tcp"); print $s "GET /files/stacksmith/postgresql-14.2.0-0-linux-amd64-debian-11.tar.gz HTTP/1.0\r\nHost: downloads.bitnami.com\r\n\r\n"; while(<$s>) { last if $_ eq "\r\n" }; open(my $f, ">", "postgresql-14.2.0-0-linux-amd64-debian-11.tar.gz") or die "Could not open file: $!"; print $f $_ while <$s>; close $f; close $s;' tar --extract --file postgresql-14.2.0-0-linux-amd64-debian-11.tar.gz;

tar --extract --directory /bitnami/postgresql/oldbin/ --file postgresql-14.2.0-0-linux-amd64-debian-11.tar.gz --strip-components=4 postgresql-14.2.0-linux-amd64-debian-11/files/postgresql/bin

# copy old bin
/opt/bitnami/scripts/postgresql/entrypoint.sh mv postgresql-14.2.0-linux-amd64-debian-11/files/postgresql/bin /bitnami/postgresql/oldbin

# copy old datadir
/opt/bitnami/scripts/postgresql/entrypoint.sh mv /bitnami/postgresql/data /bitnami/postgresql/olddata
cp /bitnami/postgresql/data/postgresql.auto.conf /bitnami/postgresql/olddata/postgresql.conf
echo "local all postgres trust" > /bitnami/postgresql/olddata/pg_hba.conf

# Initialize new cluster
/opt/bitnami/scripts/postgresql/entrypoint.sh /opt/bitnami/scripts/postgresql/setup.sh
/opt/bitnami/scripts/postgresql/entrypoint.sh chmod 700 /bitnami/postgresql/olddata
/opt/bitnami/scripts/postgresql/entrypoint.sh chmod 700 /bitnami/postgresql/data

# Create postgres.conf and trusted access file
cp /bitnami/postgresql/data/postgresql.auto.conf /bitnami/postgresql/data/postgresql.conf
echo "local all postgres trust" > /bitnami/postgresql/data/pg_hba.conf

# Check consistency fist
# If it says the versions are compatible, run the same command without -c and it will copy the data
/opt/bitnami/scripts/postgresql/entrypoint.sh pg_upgrade  -b /bitnami/postgresql/oldbin -B /opt/bitnami/postgresql/bin -d /bitnami/postgresql/olddata -D /bitnami/postgresql/data -c

mv postmaster.pid not postmaster.pid
/opt/bitnami/postgresql/tmp/postgresql.pid

# Delete pg_hba.conf and postgresql.conf so the container versions are used on reboot
rm /bitnami/postgresql/data/postgresql.conf
rm /bitnami/postgresql/data/pg_hba.conf