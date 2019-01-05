# digikeyPartGrabber
Given a Digikey Part #, pull select data from Digikey.com.

### Prerequisites

This object relies on requests and beautifulSoup

```
pip install requests
pip install beautifulSoup4
```

### Usage


```
import digikeyPartGrabber
part = digikeyPartGrabber.digikeyPart('1276-1087-1-ND')
print(part)

print(f'Value = {part.value}")
print(f"Description = {part.descripion}")
```


