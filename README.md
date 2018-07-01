# pake

[![travis](https://travis-ci.org/schollz/pake.svg?branch=master)](https://travis-ci.org/schollz/pake) 
[![go report card](https://goreportcard.com/badge/github.com/schollz/pake)](https://goreportcard.com/report/github.com/schollz/pake)
[![coverage](https://img.shields.io/badge/coverage-88%25-brightgreen.svg)](https://gocover.io/github.com/schollz/pake)
[![godocs](https://godoc.org/github.com/schollz/pake?status.svg)](https://godoc.org/github.com/schollz/pake) 

This library will help you allow two parties to generate a mutual secret key by using a weak key that is known to both beforehand (e.g. via some other channel of communication). This is a simple API for an implementation of password-authenticated key agreement (PAKE). This protocol is derived from [Dan Boneh and Victor Shoup's cryptography book](https://crypto.stanford.edu/~dabo/cryptobook/BonehShoup_0_4.pdf) (pg 789, "PAKE2 protocol). 


## Install

```
go get -u github.com/schollz/pake
```

## Usage 

![Explanation of algorithm](https://i.imgur.com/s7oQWVP.png)

```golang
// pick an elliptic curve
curve := elliptic.P224() 
// both parties should have a weak key
sharedWeakKey := []byte{1, 2, 3}
// initialize P
P, _ := Init(sharedWeakKey, 0, curve)
// initialize Q
Q, _ := Init(sharedWeakKey, 1, curve)
// P sends u to Q
Q.Update(P.Bytes())
// Q computes k, sends H(k), v back to P
P.Update(Q.Bytes())
// P computes k, H(k), sends H(k) to Q
Q.Update(P.Bytes())
// both P and Q now have session key
k := P.SessionKey()
```

The *H(k)* is a bcrypt hashed session key, which only the keeper of a real session key can verify. Passing this between P and Q allows them to understand that the other party does indeed have the session key. The session key can then be used to encrypt a message.

When passing *P* and *Q* back and forth, the structure is being marshalled using `Bytes()`, which prevents any private variables from being accessed from either party.

Each function has an error. The error become non-nil when some part of the algorithm fails verification: i.e. the points are not along the elliptic curve, or if a hash from either party is not identified. If this happens, you should abort and start a new PAKE transfer as it would have been compromised. 

## Contributing

Pull requests are welcome. Feel free to...

- Revise documentation
- Add new features
- Fix bugs
- Suggest improvements

## Thanks

Thanks [@tscholl2](https://github.com/tscholl2) for implementing the first version.

## License

MIT