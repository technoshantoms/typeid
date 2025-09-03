# UUID-compatible Typed IDs with prefixes
ref:
https://ecency.com/python/@full-steem-ahead/howto-an-introduction-to-python-bitshares
`[https://python-bitshares.readthedocs.io/en/latest/]`
https://uptick.readthedocs.io/en/latest/


[![Documentation](https://readthedocs.org/projects/typeid/badge/?version=latest)](https://typeid.readthedocs.io/en/latest/?badge=latest)

`typid` is a python library that allows to create specific id types much like
stripe uses them across their platform to distinguish different data models from
their id already. `typeid` is compatible to the UUID library and as such can act
as a drop-in replacement.

A type id has the following string representation:

    user_nnv5rr3rfk3hgry6xrygr
    └──┘ └───────────────────┘
    type    uuid (base58)

## How to use `typeid`

First generate the Types using generate(...), then use them like so:

```python
from uuid_extensions import uuid7
from typeid import generate

typeids = generate("sk", "pk", "user", "apikey")

# Now they can be used with SkId, PkId, UserId, and ApikeyId
assert str(typeids.SkId(uuid7().hex)).startswith("sk_")
assert str(typeids.PkId(uuid7().hex)).startswith("pk_")
assert str(typeids.UserId(uuid7().hex)).startswith("user_")
assert str(typeids.ApikeyId(uuid7().hex)).startswith("apikey_")

# Obviously, loading a prefixed type id in the correct Type works just fine
assert (typeids.UserId("user_nnv5rr3rfk3hgry6xrygr").hex == "0664c4135ed83faabd4bc0dc33839c9f")

# Use custom suffixes, just in case you need them
other_ids.generate("admin", suffix="IdUseWithCare")
assert str(other_ids.AdminIdUseWithCare(uuid7().hex)).startswith("admin_")
```

## SqlAlchemy Support

The library also allows to create classes specifically for the use with
sqlalchemy.

#### Example

```python
from typeid import generate

typeids = generate("user")

class User(Base):
    __tablename__ = "user"

    # use typeids.UserId in the Mapper[]
    id: Mapped[typeids.UserId] = mapped_column(
        # use typeids.UserId in the mapped_column!
        typeids.UserId, primary_key=True, default=uuid.uuid4
    )

    name: str = Column(String(128), unique=True)

user = User(name="John Doe")
(db,) = get_session()
db.add(user)
db.commit()
db.refresh(user)
print(type(user.id))
# <class 'typeid.UserId'>

print((user.id))
# user_qvnrsem3skqho2hgni2mfk
```

`PART 2`

Preliminaries / Prerequisites
I may be new to Python programming but I found it relatively easy to get started and become productive with it. If you don't have a programming or technical background it may be challenging but if you persevere, the skill you develop will serve you well for many years to come, not only for blockchain work but in many other fields as well.
I assume the reader is comfortable with obtaining private keys for his BitShares account and using computers and programming in general. The more experience you have the easier it will be. All code and discussion are for Linux only, however once you get Python installed on your platform of choice the operating system is mostly irrelevant.

The code used in this introduction is very simple. It does only one thing: monitor a witness for missed blocks and when the conditions are right uses the update_witness API call to broadcast a message on the BitShars blockchain to activate a standby witness node. I operate many full nodes, and I wanted to extend Roeland's program to make use of more than one backup node. In the future I may extend the program further so multiple switcher monitors can be used to provide even more fault tolerance and provide redundancy for the monitor itself. Aside from a text editor and a BitShares account with a balance of BTS to pay fees, the following links provide the essentials you will need:

Python-BitShares on github: http://python-bitshares.readthedocs.io/en/latest/
Uptick: http://uptick.readthedocs.io/en/latest/
Python: https://www.python.org/about/gettingstarted/

Please take note that the library's documentation is very new and although it provides much information is still lacking important details. It is mostly a collection of small code snippets and few if any complete examples. It appeared to be written quickly to capture the main points and leave details for later. Not such a good approach to encourage use. I found navigating the docs to be difficult and the search capability very limited. The docs could also be better organized. I was unable to find this page for example, which should be under the QuickStart section but isn't. This article is my offering to address some of these problems. I will provide a summary and submit it to github so xeroc can update his documentation.

Installing Python-BitShares
The first problem I had was how to install the library. I prefer to use pip to install python packages, but I could not find the proper name to use. Xeroc and others refer to the python-bitshares library by several names: PyBitShares, python-bitshares, piston, uptick, python-graphene... Not all of these refer to the same thing, and it lends to confusion. Thus my first installation attempt I downloaded a zip archive of the python-bitshares library from github and did a manual installation. Please note that the installation link I referred to above provides info on installing manually or by using pip (pip for python3 or pip3). The correct name to use when installing with pip is simply "bitshares", as in:
pip3 install bitshares
Although it isn't required, you may find the uptick program useful. The documentation for the standalone uptick program is even more sparse than for the library. It is a separate python program that works with python-bitshares and may help you create a wallet and add private keys to it for use by programs built with python-bitshares. I did use it initially but found it wasn't necessary to use with this switcher project.

Creating a Wallet
To do much of anything useful with python-bitshares you will need a BitShares account with some BTS funds in it to pay the transaction fees required by most BitShares activities. This implied requirement is not mentioned anywhere in the documentation. That seems like an obvious requirement but it didn't occur to me until after many trials and experiments. Error messages can be very misleading, for example a "MissingKeyError" when I used the update_witness API call. The actual problem wasn't a missing key but rather the one provided was not valid b/c it didn't have a BTS balance. The breakthrough came when I realized a fee is required to broadcast the update_witness API message and the private key I used was not for a funded BitShares account. So I added the Active key for my witness account. You could also use your owner key, but that is bad practice for a witness. I am not sure if a witness account is necessary, or if any bitshares account with a BTS balance could be used. The update_witness API call may be restricted to only witness accounts or Life Time Member (LTM) accounts.
I actually wrote a very short and simple program to create a new wallet and install private keys in it. You can also do that with uptick. The important point here is you need to setup a python-bitshares wallet to contain the private key for your witness. Make sure the account you use has a BTS balance. I am not sure when the wallet gets created but it took quite a while to finally figure out how to begin with a "no wallet exists" condition. Uptick does not have a "delete wallet" command either. Unless your program starts without a wallet and it creates one and adds keys, you will need some type of separate, standalone program to setup a wallet. The python-bitshares library uses a SQLite database to store various things like your private key. Not certain how secure that is, but there is one other way keys can be provided which doesn't involve the SQLite database. Consult the python-bitshares docs for more info on that. It means moving the security concern from SQLite to your self initializing program.

You can use the walletInit.py program below to create a new wallet and add a private key to it. Change the walletPassword, privateKey and a URL to a full node, then run the program. It will report whether it finds a wallet and if not it will create one. Run it again and it will add your private key to it. If you need to delete the wallet to start fresh uncomment the "os.remove" line or run the command manually: `rm -rf ~.local/share/bitshares/bitshares.sqlite`.

#!/usr/bin/env python3

from bitshares import BitShares
import os

walletPassword = "YourUltraSecretWalletPassword"
privateKey = "5K.....................................1Aw"

bitshares = BitShares("ws://localhost:port/ws")

#os.remove("/home/yourAccountName/.local/share/bitshares/bitshares.sqlite")

if bitshares.wallet.created():
    print("A wallet already exists, good. Adding your BitShares private key...")
    bitshares.wallet.unlock(walletPassword)
    bitshares.wallet.addPrivateKey(privateKey)
else:
    print("No wallet exists yet, creating one now...")
    bitshares.wallet.newWallet(walletPassword)

print("\nYou should see many values printed below on successful install,")
print("including recently_missed_count and next_maintenance_time\n")
print(bitshares.info(), "\n")
The btsNodeSwitcher.py Program
The program RoelandP wrote can be found on github here: https://github.com/roelandp/Bitshares-Witness-Monitor/blob/master/witnesshealth.py. It also includes notification functionality and you may find it more suitable to your needs. It is very simple and was easy to modify to handle more than one failover node. Here is the version I created that suits my needs:
#!/usr/bin/env python3
#
# This modified script from roelandp will monitor a witness'
# missed blocks and broadcast an update_witness msg when they
# excede the number defined by the "tresholdwitnessflip".
#
import sys
import time
from bitshares.witness import Witness
from bitshares import BitShares

# witness/delegate/producer account name to check
witness    = "YourBitSharesWitnessAcccount"   # This account must have a BTS balance to pay fees
walletpwd  = "YourUptickWalletPassword"       # Your UPTICK / bitshares.sqlite wallet unlock code
witnessurl = "https://bitsharestalk.org/"     # Your Witness Announcement url

# Array of alternate signing keys we'll switch to in round-robin fashion upon failure
backupsigningkeys = [ "BTS5...........................................qjTpc1",
                      "BTS8...........................................MNWcfX",
                      "BTS5...........................................ZtjU9P",
                      "BTS6...........................................gXZ2nU" ]

websocket       = "ws://localhost:port/ws" # API node to connect to!

next_key                = int(0)                # Index of next signing key to use when a failure is detected
check_rate              = int(30)               # Frequency in seconds between each sample of missed blocks
loopcounter             = int(0)                # Increments each time the missed block count is sampled  
startmisses             = int(-1)               # Holds number of misses (set to -1 to initialize counters)
currentmisses           = int(0)                # Current number of missed blocks according to blockchain 
counterOnLastMiss       = int(0)                # Last loopcounter value we missed a block on
resetAfterNoMisses      = int(20)               # If no missed blocks for this many samples reset counters
tresholdwitnessflip     = int(3)                # after how many blocks the witness should switch to different signing key
#currentmisses           = int(999)             # To test the update_witness call set this to current actual misses -1
#startmisses             = int(1000)            # To test set this to the current number of missed blocks
#tresholdwitnessflip     = int(0)               # To test set this to zero

# Setup node instance
bitshares = BitShares(websocket)

# Check how many blocks the witness has missed and switch signing keys if required
def check_witness():
    global counterOnLastMiss
    global lastMissedSample
    global currentmisses
    global startmisses
    global next_key
    status = Witness(witness, bitshares_instance=bitshares)
    current_key = status['signing_key']
    missed = status['total_missed']
    print("\r%d samples, missed=%d, key=%.16s..." % (loopcounter, missed, current_key), end='')

    if start misses == -1:                       # Initialize counters
        startmisses = currentmisses = missed
        counterOnLastMiss = loopcounter          # Reset periodically to prohibit switching on
                                                 #  small accumulated misses over long periods 
    if missed > currentmisses:
        print("\nMissed another block!  (%d)" % missed)
        currentmisses = missed
        counterOnLastMiss = loopcounter
        if (currentmisses - startmisses) >= tresholdwitnessflip:
            # Flip witnesses to backup
            print("Time to switch! (using key %s)" % backupsigningkeys[next_key])
            bitshares.wallet.unlock(walletpwd)
            bitshares.update_witness(witness, url=witnessurl, key=backupsigningkeys[next_key])
            time.sleep(6) # wait for 2 witness cycles
            status = Witness(witness, bitshares_instance=bitshares)
            if current_key != status['signing_key']:
                current_key = status['signing_key']
                print("Witness updated. Now using " + current_key)
                startmises = -1
                next_key = (next_key + 1) % len(backupsigningkeys)
            else:
                print("Signing key did not change! Will try again in %d seconds" % check_rate)
    else:
        # If we haven’t missed any for awhile reset counters 
        if loopcounter - counterOnLastMiss >= resetAfterNoMisses:
            startmisses = -1


# Main Loop
if __name__ == '__main__':
    if not bitshares.wallet.created():
        print("Please use uptick or your own program to create a proper wallet.")
        exit(-2)
    while True:
        check_witness()
        sys.stdout.flush()
        loopcounter += 1
        time.sleep(check_rate)
Updated on Christmas day to reset counters after 20 samples and no missed blocks.
That's about it. Thanks for dropping by!
