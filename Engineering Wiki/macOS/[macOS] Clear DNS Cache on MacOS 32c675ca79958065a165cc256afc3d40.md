# [macOS] Clear DNS Cache on MacOS

Owner: Nam Tran
Last edited time: March 23, 2026 6:32 PM

```bash
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
```