# pgbouncer
Ansible role for pgbouncer

The following vars are not set by the role defaults and should instead be set in a seperate var file (encrypted with Ansible vault or similar):

```
pgbouncer_auth_user: "pgbouncer"
pgbouncer_auth_password: "md5 hash of password prefixed with the string: md5"
pgbouncer_auth_type: "md5"
pgbouncer_auth_file: "/etc/pgbouncer/auth.conf"
pgbouncer_auth_query: "SELECT uname, phash from user_lookup($1)"
```

Note: target database should have a security definer function setup with $pgbouncer_auth_user granted access to invoke it (more info here: https://pgbouncer.github.io/config.html)

```
CREATE FUNCTION "user_lookup"("i_username" "text", OUT "uname" "text", OUT "phash" "text") RETURNS "record"
    LANGUAGE "plpgsql" SECURITY DEFINER
    AS $$
BEGIN
    SELECT usename, passwd FROM pg_catalog.pg_shadow
    WHERE usename = i_username INTO uname, phash;
    RETURN;
END;
$$;

GRANT ALL ON FUNCTION "user_lookup"("i_username" "text", OUT "uname" "text", OUT "phash" "text") TO "pgbouncer";
```

Database targets for pgbouncer to connect to are specified as a list using the pgbouncer_databases var. Example:

```
pgbouncer_databases:
  - { name: 'mydb', host: '127.0.0.1', dbname: 'mydb', encoding: '{{ pgbouncer_default_client_encoding }}', auth_user: '{{ pgbouncer_auth_user }}', comment: 'My db' }
```
