# Ansible Role: pgbouncer

Ansible role to compile and install pgbouncer (a reverse proxy for postgresql).

The following vars should be set in a separate var file and encrypted with Ansible vault (or similar):

```
pgbouncer_auth_user: "pgbouncer"
pgbouncer_auth_password: "md5 hash of password prefixed with the string: md5"
pgbouncer_auth_type: "md5"
pgbouncer_auth_file: "/etc/pgbouncer/auth.conf"
pgbouncer_auth_query: "SELECT uname, phash from user_lookup($1)"
```

Databases for pgbouncer to connect to are specified as a list using the pgbouncer_databases var. Example:

```
pgbouncer_databases:
  - { name: 'mydb', host: '127.0.0.1', dbname: 'mydb', encoding: '{{ pgbouncer_default_client_encoding }}', auth_user: '{{ pgbouncer_auth_user }}', comment: 'My db' }
```

## Dependencies

Target postgres databases must have a security definer function setup with $pgbouncer_auth_user granted access to invoke it (more info here: https://pgbouncer.github.io/config.html):

```
CREATE SCHEMA pgbouncer AUTHORIZATION pgbouncer;
CREATE OR REPLACE FUNCTION pgbouncer.user_lookup(in i_username text, out uname text, out phash text)
RETURNS record AS $$
BEGIN
    SELECT usename, passwd FROM pg_catalog.pg_shadow
    WHERE usename = i_username INTO uname, phash;
    RETURN;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
REVOKE ALL ON FUNCTION pgbouncer.user_lookup(text) FROM public, pgbouncer;
GRANT EXECUTE ON FUNCTION pgbouncer.user_lookup(text) TO pgbouncer;
```

## License

MIT / BSD
