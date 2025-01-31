`powerbox-http-proxy` is a daemon that is sometimes helpful for
porting legacy applications that want HTTP access to [Sandstorm][1]. It
acts as an HTTP proxy, and makes powerbox requests on-demand for domains
that the legacy application tries to access. It is used by the [Tiny
Tiny RSS Port](https://github.com/zenhack/ttrss-sandstorm)

# Building

Install a recent Go toolchain (new enough to support Go modules) and
npm. Then run:

```
go build
npm install
npm run build
```

This will create two files, an executable `powerbox-http-proxy` and a
file `build/index.js`

# Configuration

To use `powerbox-http-proxy` with your application, you need to do the
following:

- Build the daemon and its front-end JavaScript helper, per the above
  instructions.
- Add a `<script>` tag to your pages which includes the generated
  `build/index.js`.
- Arrange for the url `/_sandstorm/websocket` to be served by the
  daemon; see below.
- Arrange for your application to trust a root TLS cert generated by
  the daemon; this is necessary to proxy HTTPS requests.

The daemon is mainly configured through environment variables:

- `POWERBOX_PROXY_PORT` specifies the port that the daemon should listen
  on for proxy requests; you should configure your application to use
  `http://localhost:$POWERBOX_PROXY_PORT` as its proxy for both http
  and https. Many applications will observe the variables `http_proxy`
  and `https_proxy`, so you can often make this happen by just defining
  those variables.
- `POWERBOX_WEBSOCKET_PORT` specifies the port to listen on for connections
  from the JavaScript helper. You will want to configure your web server
  to point requests to `/_sandstorm/websocket` to this port. For example,
  for nginx, assuming `POWERBOX_WEBSOCKE_PORT` is `3000`, you can use:  

        location /_sandstorm/websocket {
          proxy_pass http://127.0.0.1:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "Upgrade";
          proxy_set_header Host $host;
        }        
        
- `CA_CERT_PATH` is the path to which the daemon should write its root
  certificate. Configure your application to trust the cert written
  to this location.
- `DB_TYPE`: The type of the database to use. Supported values are:
   - `mysql` for MySQL/MariaDB
   - `sqlite3` for SQLite.
- `DB_URI`: The location of a mysql database in which to store the
  daemon's private data. The format depends on the value of DB_TYPE.
  - For SQLite, this should be the path to the database file.
  - For MySQL, see
    <https://github.com/go-sql-driver/mysql#dsn-data-source-name> for
    a description of the format.

[1]: https://sandstorm.io
