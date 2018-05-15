# How Addresses pass from the Web into M3

There are quite a few intricacies related to how addresses pass from web into M3. This relates to the order staging process, and there are many specific requirements which must be understood. Among these useful bits of knowledge are:

## AddressID really means AddressType and has only three possible values

These values and their meanings are:

- -1: New address
- 0: Default address
- 1: A pre-existing non-default address

## M3AddressID must start with either an I or a D

- I stands for Invoice and is used for billing.
- D stands for Delivery and is used for shipping.

## Additional factoids

M3 addresses can be duplicated. There is no way to prevent this, however the web will attempt to filter out duplicates.
The web has to perform the M3AddressID incrementation however, and should be careful to not choos the same number as an M3AddressID that had previously been filtered out.

The current default M3AddressID is **D000** or **I000**, however, this may be updated to **D001** or **I001**
