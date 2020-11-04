# ASN.1

## IMPLICIT and EXPLICIT tags

**参考资料：**
* [EXPLICIT vs IMPLICIT tags in ASN.1](https://groups.google.com/forum/#!topic/sci.crypt/oTBrXmWrvIY)
* [Wikipedia BER Encoding](https://en.wikipedia.org/wiki/X.690#BER_encoding)
* [ASN.1 Tagging](http://powerasn.ncottin.net/download/ASN1_Tagging.pdf)

**问题描述：**

看X.509协议文档时看到TBSCertificate的定义中出现了IMPLICIT tag，如下：
```text
TBSCertificate  ::=  SEQUENCE  {
    version         [0]  EXPLICIT Version DEFAULT v1,
    serialNumber         CertificateSerialNumber,
    signature            AlgorithmIdentifier,
    issuer               Name,
    validity             Validity,
    subject              Name,
    subjectPublicKeyInfo SubjectPublicKeyInfo,
    issuerUniqueID  [1]  IMPLICIT UniqueIdentifier OPTIONAL,
                         -- If present, version shall be v2 or v3
    subjectUniqueID [2]  IMPLICIT UniqueIdentifier OPTIONAL,
                         -- If present, version shall be v2 or v3
    extensions      [3]  EXPLICIT Extensions OPTIONAL
                         -- If present, version shall be v3
    }

UniqueIdentifier  ::=  BIT STRING


Extension  ::=  SEQUENCE  {
    extnID      OBJECT IDENTIFIER,
    critical    BOOLEAN DEFAULT FALSE,
    extnValue   OCTET STRING  }
```
那么IMPLICIT和EXPLICIT tag的作用是什么？对于编码有怎样的影响呢？

**作用**

IMPLICIT和EXPLICIT tag的主要作用是避免编码的二义性。在上面的例子中，issuerUniqueID和subjectUniqueID的类型都是UniqueIdentifier,
而且都是Optional的。因此，如果证书中只出现issuerUniqueID和subjectUniqueID其中之一，那么解码的一方将无法从编码上区分。

为了区分，规范在制定的时候可以使用IMPLICIT或者EXPLICIT tag给可能出现二义性的类型加上一个ID。在编码的时候，这个ID会被加进去，而解码的时候，
解码器可以根据这个ID来确定某个字段的实际含义。

**如何编码**

IMPLICT和EXPLICIT在编码的时候略有不同：
* IMPLICIT: 被编码成CONTEXT类型的TLV结构。CONTEXT的Tag number就是[ ]里的Tag值，而字段是实际类型则不起作用。比如issuerUniqueID
会被编码成CONTEXT-PRI-1，而不是BITSTRING。CONTEXT的V所包含的是issuerUniqueID的实际数值。
* EXPLICIT: 被编码成CONTEXT类型的TLV结构。CONTEXT的Tag number就是[ ]里的Tag值，而V则成为字段的实际类型编码的一个Wrapper。简单
说来EXPLICIT编码下，它会变成一个TL(TLV)结构。内层的TLV其实是BITSTRING的TLV结构。

**总结：**

IMPLICT会去掉内层的TLV结构，而EXPLICIT则保留内层的TLV结构。