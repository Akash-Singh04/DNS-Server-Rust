# DNS_Server-Rust
Building a DNS Server from Scratch in RUST!


## Part 1 : Implementing the protocol

DNS Packets ko bhejte h over UDP transport and limited to 512 bytes(Wo alag baat h there is exception where it can be sent over TCP as well and eDNS se packet size badha sakte h).

DNS uses the same format in queries and responses. Mainly a DNS packet consists of:
- Header : Isme hoga information about the query/response
- Question Section : List of questions(hota ek hi h in practice) , each indicating the query name(domain) and the record type of interest
- Answer Section : List of relevant records of the requested type
- Authority Section : A list of name servers used for resolving queries recursively.
- Additional Section : Additional useful info.

Essentially 3 different objects ko support karna hoga:
- Header : [DNSHeader.rs](Protocol/DNSHeader.rs) mein implement kar diye : Iske liye we created another implementaion for the rescode field also in [ResultCode.rs](Protocol/ResultCode.rs). RCode is set by the server to indicate the status of the response, i.e. whether or not it was successful or failed, and agar fail hua toh providing details about the cause of the failure.
- Question : [DNSQuestion.rs](Protocol/DNSQuestion.rs) : Iske liye we created another implementation of [QueryType](Protocol/QueryType.rs), so that we can represent the *record type* being queried. 
- Record : [DNSRecord.rs](Protocol/DNSRecord.rs) is used to represent the actual dns records and allow us to add new records later on easily.


[BytePacketBuffer.rs](Protocol/BytePacketBuffer.rs) asli problematic kaam karta h. The thing is DNS encodes each name into a sequence of labels, with each label prepended by a single byte indicating its length. Example would be *[3]www[6]google[3]com[0]*. Ye phir bhi theek h, but it gets even more problematic when jumps come into place. 

> Due to the original size constraints of DNS, of 512 bytes for a single packet, some type of compression was needed. Since most of the space required is for the domain names, and part of the same name tends to reoccur, there's some obvious space saving opportunity.

To save space from reoccuring set of characters, DNS packets include a "jump directive", telling the packet parser to jump to another position, and finish reading the name there. This _jump_ can be read if the length byte has the two most significant bits set,iska matlab jump hai, and we need to follow the pointer.

Ek aur baat to be taken care of is, this jumps can cause a cycle if some problematic person adds it to the packet, so wo check karna padega. This along with reading of the packets is all implemented in the BytePacketBuffer.

Bas phir, we can put together all of this now in our [DNSPacket.rs](Protocol/DNSPacket.rs) to finish our protocol implementation.

To test it out, run the [main.rs](Protocol/main.rs) file with our `response_packet.txt` 