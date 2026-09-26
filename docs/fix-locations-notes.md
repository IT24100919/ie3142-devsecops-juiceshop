Login route (SQLi target): routes/login.ts, routes/saveLoginIp.ts
Search route (XSS target): routes/search.ts
Basket route (IDOR target): routes/basket.ts, routes/basketItems.ts
FTP/file exposure route: server.ts (lines 287-290), routes/fileServer.ts

Note: server.ts explicitly marks the /ftp directory listing as an
intentional vulnerability via code comments (vuln-code-snippet
directoryListingChallenge), confirming this is a documented CWE-mapped
flaw rather than an accidental one.
