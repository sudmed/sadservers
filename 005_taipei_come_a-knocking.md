# "Taipei": Come a-knocking

**Description:** There is a web server on port `:80` protected with [Port Knocking](https://en.wikipedia.org/wiki/Port_knocking). Find the one "knock" needed (sending a SYN to a single port, not a sequence) so you can curl localhost.  

**Test:** Executing curl localhost returns a message with `md5sum fe474f8e1c29e9f412ed3b726369ab65`. (Note: the resulting md5sum includes the new line terminator: `echo $(curl localhost)`)  

---

### Solution:
#### 1. Reconnaissance on the server
