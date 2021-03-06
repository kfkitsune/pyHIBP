pyHIBP (pyHave I Been Pwned)
============================
[![image](https://img.shields.io/pypi/v/pyHIBP.svg)](https://pypi.org/project/pyHIBP/)
[![image](https://img.shields.io/pypi/l/pyHIBP.svg)](https://pypi.org/project/pyHIBP/)
[![image](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gl/kitsunix%2FpyHIBP%2FpyHIBP-binder/master)


A Python interface to Troy Hunt's 'Have I Been Pwned?' (HIBP) public API. A full reference to the API
specification can be found at the [HIBP API Reference](https://haveibeenpwned.com/API/v3).

This module detects when the rate limit of the API has been hit, and raises a RuntimeError when the limit
is exceeded, or when another API-defined error condition is encountered based on the submitted data. When
data is found from a call, the data returned will be in the format as retrieved from the endpoint, documented
in the return-type information for the relevant function.

Note that the `pwnedpasswords` API backend does not have a rate limit. If you are intending to bulk-query passwords or
hashes, you should consider downloading the raw data files accessible via the [Pwned Passwords](https://haveibeenpwned.com/Passwords) page.

Installing
----------
```bash
$ pip install pyhibp
```

Example usage
-------------
For an interactive example, check out the Jupyter Notebook for [`pyhibp`](https://mybinder.org/v2/gl/kitsunix%2FpyHIBP%2FpyHIBP-binder/master?filepath=/pyHIBP.ipynb),
as well as [`pyhibp.pwnedpasswords`](https://mybinder.org/v2/gl/kitsunix%2FpyHIBP%2FpyHIBP-binder/master?filepath=/pyHIBP.pwnedpasswords.ipynb).

```python
import pyhibp
from pyhibp import pwnedpasswords as pw

# Required: A descriptive user agent must be set describing the application consuming
#   the HIBP API
pyhibp.set_user_agent(ua="Awesome application/0.0.1 (An awesome description)")

# Check a password to see if it has been disclosed in a public breach corpus
resp = pw.is_password_breached(password="secret")
if resp:
    print("Password breached!")
    print("This password was used {0} time(s) before.".format(resp))

# Get data classes in the HIBP system
resp = pyhibp.get_data_classes()

# Get all breach information
resp = pyhibp.get_all_breaches()

# Get a single breach
resp = pyhibp.get_single_breach(breach_name="Adobe")

# An API key is required for calls which search by email address
#   (so get_pastes/get_account_breaches)
# See <https://haveibeenpwned.com/API/Key>
HIBP_API_KEY = None

if HIBP_API_KEY:
    # Set the API key prior to using the functions which require it.
    pyhibp.set_api_key(key=HIBP_API_KEY)

    # Get pastes affecting a given email address
    resp = pyhibp.get_pastes(email_address="test@example.com")

    # Get breaches that affect a given account
    resp = pyhibp.get_account_breaches(account="test@example.com", truncate_response=True)
```

Developing
----------
In order to ensure we have a consistent and repeatable development environment
we use a virtual environment, namely `pipenv`.
To develop or test, execute the following:

```bash
# Install the prerequisite virtual environment provider
$ pip install pipenv
# Initialize the pipenv environment and install the module within it
$ make dev
# To run PEP8, tests, and check the manifest
$ make tox
```

Other commands can be found in the `Makefile`.

Goals
-----
- Synchronize to the latest HIBP API(s), implementing endpoint accessing functions where it makes sense. For instance,
  in the interest of security, the ability to submit a SHA-1 to the Pwned Passwords endpoint is not implemented. See
  "Regarding password checking" below for further details.
- For breaches and pastes, act as an intermediary; return the JSON as received from the service.

Regarding password checking
---------------------------
- For passwords, the option to supply a plaintext password to check is provided as an implementation convenience.
- For added security, `pwnedpasswords.is_password_breached()` only transmits the first five characters of the SHA-1
  hash to the Pwned Passwords API endpoint; a secure password will remain secure without disclosing the full hash.
