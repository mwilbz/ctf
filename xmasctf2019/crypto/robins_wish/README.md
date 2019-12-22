# Problem

> Santa tried using public key cryptography to secure his letters, but his plan turned out to be a failure! After deleting his private keys, one of his elves found an encrypted letter from a child named Robin. Asking Robin what he wants for Christmas would be unprofessional, so the elf asked you to decrypt the letter. He also gave you access to a more basic copy of Santa's service.
> 
> Help Santa decrypt Robin's letter and you will definitely not be on the 'naughty' list this year!
>
> Remote server: nc challs.xmas.htsp.ro 10005

# Resources

- [Rabin Cryptosystem](https://en.wikipedia.org/wiki/Rabin_cryptosystem#Encryption_Algorithm)

# Solution

## TLDR

Find `n` and `e`, realize `e=2` means it's not RSA. Find Rabin's Cryptosystem and decrypt.

## Gathering Information

Connecting to the provided server shows the following prompt:

```
$ nc challs.xmas.htsp.ro 10005
  __          ___       __ 
 (_   /\  |\ | |  /\ / (_  
 __) /--\ | \| | /--\  __) 
                           
     _ ___ ___ _  _        
 |  |_  |   | |_ |_)       
 |_ |_  |   | |_ | \       
                           
  __  _  _      ___  _  _  
 (_  |_ |_) \  / |  /  |_  
 __) |_ | \  \/ _|_ \_ |_  
                           

Welcome to Santa's letter encryption service!

Enter your letter here and get an encrypted version which only Santa can read.
Note: Hackers beware: are using military-grade encryption!
Menu:
1. Encrypt a letter
2. Encrypt your favourite number
3. Exit
1337. Get Robin's Encrypted Letter (added by elf)
```

By playing around with encrypted numbers you quickly find small inputs are being squared:

```
Choice: 2
Your favourite number: 10
You can share your favourite number by sending him the following number: 100
```

But sufficiently large inputs are getting reduced modulo some number `n`:

```
Your favourite number: 1000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
You can share your favourite number by sending him the following number: 5245011011602491803664370575472709624274548272259320091433809466021203426817310186896310029658155776937528355323344006923977113553826210044716916761943666682385319625490349573808126763817318295478525328773318638213052114802345748146506404691689995405503303822016515046458252373127745904567619723597957297946422902129529543502645693599861443318116217613781848013941342646655217928664426579486750104198510693970393278733645664236923097146527594991807640347114720428789174163906930670960136912483150762248581418744755856956008035040747559729722485249549016117749161435076726496078746126920044963596268775441573974424006
```

There's several ways to find the value of `n` but I wrote a binary search:

```
def search():
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect(('challs.xmas.htsp.ro', 10005))
	s.recv(1024)

	high = gmpy2.mpz(10**309)
	low = gmpy2.mpz(10**308)
	while high > low:
		print('Low: ', low)
		print('High:', high)
		print()
		mid = (high + low) // 2
		s.send(b'2\n')
		s.recv(1024)
		s.send(str(mid).encode('ascii') + b'\n')
		response = s.recv(1024)
		first_line = codecs.decode(response, 'ascii').split('\n')[0]
		encrypted = int(first_line.split(' ')[-1])
		if gmpy2.square(mid) == encrypted:
			low = mid
		else:
			high = mid
```

Once it's converged, calculate `n = high**2 - cipher(high)`. `n` happened to be

```
17150948086006853589591993610767711903029749167719666894975279147137565458158322238156960171902445590052801235253045792984069360111140927413022122124794074712372666903008787076313652986830735891457266804676322092444602549744787142273336096470832931113698218899620404912992099097015038863714351384075897287966440984446042594077540591489657561322101444523900312965276873402643875552954061610698504308548301539759131150366661281651087532807818489741520557925049746199503634928208501195328273501508911193754334803125090416259379171809642283452935819219835361791073290320084884025929676790915171638558685021113076310785793
```

which factors into two primes (found on FactorDB) as you'd expect with RSA, but `e=2` has no inverse mod `lambda(n) = (p-1)(q-1)`. For a while I read too much into the description and assumed it was incorrectly implemented RSA, but a hint from an organizer pointed me to look elsewhere. Searching something like "squared RSA cipher" in Google found [Rabin's Cryptosystem](https://en.wikipedia.org/wiki/Rabin_cryptosystem), which lets you decrypt the message.

Flag: `X-MAS{4cTu4lLy_15s_Sp3lLeD_r4b1n_n07_r0b1n_69316497123aaed43fc0}`
