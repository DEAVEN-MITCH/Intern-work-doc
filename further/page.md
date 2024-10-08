---
description: Simple Binary Encoding
---

# SBE

Simple Binary Encoding (SBE) is one of the possible syntaxes for FIX messages, but not limited to FIX messages. The scope comprises the encoding (wire format) and the message schema for SBE.

{% embed url="https://github.com/real-logic/simple-binary-encoding?tab=readme-ov-file" %}

{% embed url="https://github.com/FIXTradingCommunity/fix-simple-binary-encoding/blob/master/v1-0-STANDARD/doc/01Introduction.md" %}

SBE was designed to meet the needs of high performance trading systems, but it may also be applicable to other applications with similar performance characteristics. It is optimized for low latency of encoding and decoding while keeping bandwidth utilization reasonably small. For compatibility, it is intended to represent all FIX semantics.

This encoding specification describes the wire protocol for messages. Thus, it provides a standard for interoperability between communicating parties. Users are free to implement the standard in a way that best suits their needs.

The encoding standard is complementary to other FIX standards for session protocol and application level behavior.

The binary type system has been enhanced in these ways:

* Provides a means to specify precision of decimal numbers and timestamps, as well as valid ranges of numbers.
* Differentiates fixed-length character arrays from variable-length strings. Allows a way to specify the minimum and maximum length of strings that an application can accept.
* Provides a consistent system of enumerations, Boolean switches and multiple-choice fields.

### Design principles

The message design strives for direct data access without complex transformations or conditional logic. This is achieved by:

* Usage of native binary datatypes and simple types derived from native binaries, such as prices and timestamps.
* Preference for fixed positions and fixed length fields, supporting direct access to data and avoiding the need for management of heaps of variable-length elements which must be sequentially processed.

Field

A field is a unit of data contained by a FIX message. Every field has the following aspects: semantic data type, encoding, and metadata. They will be specified in more detail in the sections on data type encoding and message schema but are introduced here as an overview.

Required and optional fields of the same primitive type have the same data range. The null value must not be set for a required field.

2.3.2 Non-FIX types Encodings may be added to SBE messages that do not correspond to listed FIX data types. In that case, the encoding and fields that use the encoding will not have a semanticType attribute.

Integer encoding

Primitive type encodings

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

默认小端，最低位在地址低位

Decimal encoding分为尾数和指数，指数以10为底

将指数设为负常数获得定点数

可能有字节对齐。

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

浮点数，遵从IEEE

For both float and double precision encodings, null value of an optional field is represented by the Not a-Number format (NaN) of the standard encoding. Technically, it indicated by the so-called quiet NaN.

char

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Valid values of a char field are printable characters of the US-ASCII character set (codes 20 to 7E hex.) The implicit nullValue is the NUL control character (code 0).

字符串

A length attribute set to zero indicates variable length. See section 2.7.3 below for variable-length data encoding.

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

If a field is required, both the Length and data fields must be set to a "required" attribute.

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Values are distinguished by position in the field. Year and month must always be populated for a non null field. Day and week are set to special value indicating null if not present. If Year is set to the null value, then the entire field is considered null.

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

Time zone is represented as an offset from UTC in the ISO 8601:2004 format ±hhmm.

4.13 Enumeration encoding An enumeration conveys a single choice of mutually exclusive valid values.

Enumerations of other datatypes, such as String valid values specified in FIX, should be mapped to an integer wire format in SBE.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

A Boolean field is a special enumeration with predefined valid values: true and false. Like a standard enumeration, an optional Boolean field may have nullValue that indicates that the field is null (or not applicable).

multi-value多值选择

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

Note that the framing standard specifies that the framing header will always be encoded in big endian byte order, also known as network byte order.大端法传送帧头部

<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

&#x20;Block length only represents message body fields; it does not include the length of the message header itself, which is a fixed size.

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

padding 可通过offset属性实现

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Note that padding will only result in deterministic alignment if the repeating group contains no variable-length fields.组和消息的尾部填充可以通过blockLength实现，在不考虑变长字段的情况下。

Each group is associated with a required counter field of semantic data type NumInGroup to tell how many entries are contained by a message. The value of the counter is a non-negative integer. See section Encoding of repeating group dimensions for details.

The space reserved for all entries of a group is the product of the space reserved for each entry times the value of the associated NumInGroup counter. If the counter field is set to zero, then no entries are sent in the message, and no space is reserved for entries. The group dimensions including the zero-value counter is still transmitted, however.零元素组的组维度仍然会被传输，而条目不占空间。

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Nested repeating groups are encoded on the wire by a depth-first walk of the data hierarchy. For example, all inner entries under the first outer entry must be encoded before encoding outer entry 2. (This is the same element order as FIX tag=value encoding.)解码时无法直接地址访问，必须挨个访问得到数量信息。

If a group contains nested repeating groups, then a NumInGroup counter of zero implies that both that group and its child groups are empty. In that case, no NumInGroup is encoded on the wire for the child groups.空组的子组信息不存在。

By default, the name of the group dimension encoding is groupSizeEncoding. This name may be overridden by setting the dimensionType attribute of a element.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption><p>xsd定义消息体的成员顺序</p></figcaption></figure>

内嵌组的顺序和消息体的顺序相同，递归定义

The attribute semanticType must be specified on either a field or on its corresponding type encoding. It need not be specified in both places, but if it is, the two values must match.

复合类型可以内嵌复合类型。

For a composite type, nullness is indicated by the value of its first element. For example, if a price field is optional, a null value in its mantissa element indicates that the price is null.

A composite type often has its elements defined in-line within the XML element as shown in the example above. Alternatively, a common type may be defined once on its own, and then referred to by name with the composite type using a element.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

An enumeration explicitly lists the valid values of a data domain. Any number of fields may share the same enumeration.

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To define a message type, add a element to the root element of the XML document, . The name and id attributes are required. The first is a display name for a message, while the latter is a unique numeric identifier, commonly called template ID.

By default, message size is the sum of its field lengths. However, a larger size may be reserved by setting blockLength, either to allow for future growth or for desired byte alignment. If so, the extra reserved space should be filled with zeros by message encoders.

Note that there need not be a one-to-one relationship between message template (identified by id attribute) and semanticType attribute. You might design multiple templates for the same FIX MsgType to optimize different scenarios.

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>



{% embed url="https://github.com/FIXTradingCommunity/fix-simple-binary-encoding/blob/master/v1-0-STANDARD/doc/publication/Simple%20Binary%20Encoding%20-%20Technical%20Specification%20version1-0.pdf" %}
English version of pdf document
{% endembed %}

## SBE接口

{% embed url="https://github.com/binance/binance-spot-api-docs/blob/master/sbe/schemas/spot_2_0.xml" %}
最新的sbe shema
{% endembed %}

{% embed url="https://github.com/binance/binance-sbe-cpp-sample-app?tab=readme-ov-file" %}
示例sbe接口
{% endembed %}

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

simple-binary-encoding的代码生成器直接根据xml生成对应的代码，用于解析sbe流。

## 变故

由于biance目前只支持现货的sbe response，与期货报单的需求不匹配，所以放弃。
