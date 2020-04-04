# iota-iot-analysis
(This repo is working progress)

## CONFLUENT

Self Managed Platform -> https://www.confluent.io/download

Create Topic: iota-gateway with value schema: 

```json
{
  "type": "record",
  "namespace": "org.fabryprog.iota.kafka.pojo",
  "name": "Transaction",
  "version": "1",
  "fields": [
    { "name": "hash", "type": "string", "doc": "Transaction Hash" },
    { "name": "address", "type": "string", "doc": "Address" },
    { "name": "value", "type": "long", "doc": "Transaction value" },
    { "name": "tag", "type": "string", "doc": "TAG" },
    { "name": "timestamp", "type": "long", "doc": "Timestamp" },
    { "name": "payload", "type": "string", "doc": "Payload" }
  ]
}
```

## GATEWAY

IOTA to KAFKA -> https://github.com/Fabryprog/iota-kafka-gateway

Settings:
 - topic -> "iota-gateway"
 - iota zmq node -> "tcp://ultranode.iotatoken.nl:5556"

## KSQL

Create STREAM **IOTA**

```sql
CREATE STREAM IOTA
 WITH (KAFKA_TOPIC='iota-gateway',
       VALUE_FORMAT='AVRO',
       KEY='hash');
```

Create STREAM **IOTA_JSON** to filter JSON data

```sql
CREATE STREAM IOTA_JSON
 AS SELECT * FROM IOTA WHERE SUBSTRING(payload,1,1) = '{'
```

Analisys: group by first 20 characters

```sql
SELECT SUBSTRING(payload,1,20), COUNT(*) FROM IOTA_JSON GROUP BY SUBSTRING(payload,1,20)
EMIT CHANGES;
```
Look at the output, today there are 4 types of JSON with following first JSON element:
 - id
 - v
 - device
 - type
 - channel
 - TanglePigeon
 
N.B. I know! I know! it is very simplistic method. Do you have another idea? :-)

CREATE A STREAM for every JSON flow

```sql
CREATE STREAM IOTA_JSON_ID
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.id') IS NOT NULL);

CREATE STREAM IOTA_JSON_V
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.v') IS NOT NULL);

CREATE STREAM IOTA_JSON_DEVICE
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.device') IS NOT NULL);

CREATE STREAM IOTA_JSON_TYPE
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.type') IS NOT NULL);

CREATE STREAM IOTA_JSON_CHANNEL
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.channel') IS NOT NULL);

CREATE STREAM IOTA_JSON_TANGLEPIGEON
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.TanglePigeon') IS NOT NULL);

```

## Retrieve data locally

Now we must be able to retrieve data from kafka stream. 

There are some clients, one of them called **kafka-cat** (https://github.com/edenhill/kafkacat). It's a simple CLI to produce or consume kafka topics using the terminal. 
(Note: kafka-cat could be install or use with docker)

I will use it with docker:

```
 docker run -it --network=host edenhill/kafkacat:1.5.0 kafkacat -C -b MY_BROKER_IP:9092 -t IOTA_JSON_CHANNEL -s avro -r http://MY_REGISTRY_IP:8081
```

Output is very pretty. See below for an real example:

```
{"HASH": {"string": "I9VSEMPWYQVFCMZKMIPIJJCDGQBBGWJHEKLKMMLKR9EUAD9YPDMTHMWH9OLX9LKAVGBKPNSWDRGJZ9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585910409}, "PAYLOAD": {"string": "{\"channel\": \"TEST\", \"text\": \"PING DGNXYPFPJOJI9CCQ9FUVJPT9OIFWYGIGLHTADUOIDVARHVWNGZXBSRWLVHMXEJNSWITXLAMUBOIJLUDYE\"}"}}
{"HASH": {"string": "MBKCNV99EUIJWYCE9ELQX9FJIGIEPYYNSWUHMUBGEQUVJSNVT9EGSOLHFJVIQWASNHHTTXGMFKIEZ9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585910461}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"Coronavirus: Nightingale Hospital opening at London's ExCel centre - Prince Charles is opening the NHS Nightingale hospital at the ExCel centre via video-link. - https://www.bbc.co.uk/news/uk-52150598\"}"}}
{"HASH": {"string": "NCM9BO9HNGFYUKANIOJFF9MJVYIZDXJCSJFVSWQGQUWHJRYDMURSZVABPHQDMEVON9BNHZLKFOKF99999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585910464}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"Dr William Frankland, allergy scientist pioneer, dies aged 108 - Dr William Frankland, known as 'the grandfather of allergy', developed the idea of a pollen count. - https://www.bbc.co.uk/news/uk-52150361\"}"}}
{"HASH": {"string": "VESMSOTRMOQWANCPDYXYCCQWXDXB9VIOZFPEDATD9NQT99CNEVDCD9YXMCTYCGOXDLXS9RPMKLCLA9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585910469}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"Solomon Islands: Dozens missing after ferry defies cyclone warning - A ferry in the Solomon Islands embarks despite the pacific nation being on alert over a cyclone. - https://www.bbc.co.uk/news/world-asia-52149915\"}"}}
{"HASH": {"string": "KDNVOMFTYXQRRSCVORWZKWLPTVTWRRNKOBNWNHZIAEBNTKPTSDZC9MVTWIFBCAO9ZQBLTESZODID99999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585911004}, "PAYLOAD": {"string": "{\"channel\": \"TEST\", \"text\": \"PING DJWOZQIXDHNLEQFZMQUBQXAWKPQZEXTROCXYANBIFDKHALASUTLVAYUSDYMJKYJRPRSDGBVWEDCHMQHFI\"}"}}
{"HASH": {"string": "ANPGKVIZLCITIEXVAMWSTDCKIMFRTYP9VVJPVGSEUYZBAGFDSAQ9PPMROIEKJLDZZBSVIECINBOLZ9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585925402}, "PAYLOAD": {"string": "{\"channel\": \"TEST\", \"text\": \"PING RDTEZMCBWGKLIMFWFOFEINXXMQACXMNAWASYKHYGEKRBSITBGPFTZHOKVRJNZOYHHALNHVXJVAT9RSVGP\"}"}}
{"HASH": {"string": "ZNOSA9PZWMSXIIRJNZNURAYGHOILZGJHWHPCMO9GBPUIFVO9QMNQCU9YZIHOENRMODHYSTVWCKJA99999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585925461}, "PAYLOAD": {"string": "{\"channel\": \"TWEETS\", \"text\": \"{\\\"text\\\":\\\"Introduction to #Coordicide Specifications - a series of 8 educational videos by our Research team first developed\\u2026 https://t.co/9B0zPnn0m0\\\", \\\"url\\\":\\\"https://twitter.com/IoTify_org/status/1246087095426433024\\\"}\"}"}}
{"HASH": {"string": "SXZUENSGMRZEWAZMJPSNQZ9QARXVLVXZ9GHYXXUMOPBRCUBAGNDFWMOVORVQVAMHMFEPMTJFEZXXZ9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585925464}, "PAYLOAD": {"string": "{\"channel\": \"TWEETS\", \"text\": \"{\\\"text\\\":\\\"@TheAutonomousT1 @PieterJoe1 @whayliang @CryptoLiveLeak @iotatoken @abbcfoundation I am afraid #IOTA will lose this\\u2026 https://t.co/oFE3yn8HRQ\\\", \\\"url\\\":\\\"https://twitter.com/78Xantus/status/1246086362580815878\\\"}\"}"}}
{"HASH": {"string": "YTTOZYDJZ9WPUXADDLLUKPLFDVWYYRVVUMUXGEMFQC9KPCJUVYKJTHFROSMM9KYCNCXZOWOSCDAGZ9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585925467}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"Coronavirus: Nurse Areema Nasreen dies with Covid-19 - The mother-of-three died at Walsall Manor Hospital, while a second nurse has also died in Kent. - https://www.bbc.co.uk/news/uk-england-birmingham-51952607\"}"}}
{"HASH": {"string": "RXDNEXK9RQLOXETTJDNJJWIOECFLMTVADYSKBJIWTXQRAZQGRXHKIFUWIGZJLEUKVFIIQ9VJP9DC99999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585925487}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"Coronavirus: Debenhams set to appoint administrators - The department store chain says it is 'making contingency plans' amid the Covid-19 outbreak. - https://www.bbc.co.uk/news/business-52156457\"}"}}
{"HASH": {"string": "PBZIOBG9LYZIOX9KOJUBXCGPIEXWDAFGRVXKHZUYEFASXMVNWIC9WSTWGNJMK9FDHQARMAUUSPFV99999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585925489}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"Premier League clubs to ask players to take 30% wage cut - Premier League clubs will ask players to take a 30% wage cut as it is announced the season will not resume from 30 April. - https://www.bbc.co.uk/sport/football/52148955\"}"}}
{"HASH": {"string": "QASTBZNM9OQPIXVRADMMVBYXBCCMMUGSRJSHHB9PCXNZFKFAIGVJYNQQQRARJGOUWY9EOWIIYOJJZ9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585926004}, "PAYLOAD": {"string": "{\"channel\": \"TEST\", \"text\": \"PING KLSJX9CFTWTA9ZBOWKLZARYLOLUICMMYJMWDIXBSNSMAMVAZ9YFRF9OXNZIYGNFRQGCTGZWC9SJLXLGVQ\"}"}}
{"HASH": {"string": "ZBDSCVCDXXJWZXMRUQ9MZFWSJZAH9S9MIDVJYJNO9NUZGJZRNJWDHMMKHBW9YFOLLSEXENXZJHXU99999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585926062}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"Premier League to ask players to take 30% wage cut - Premier League clubs will ask players to take a 30% wage cut as it is announced the season will not resume from 30 April. - https://www.bbc.co.uk/sport/football/52148955\"}"}}
{"HASH": {"string": "DRBSDMGELUBFJISACIVQKCVXXXIXXGX9CV99HNKSOESK9KXWVPGFXELHNZKUETTTGSAQVSYQDBQWA9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585926065}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"BBC Sport looks back at the time Kylian Mbappe faced Argentina at World Cup 2018. - Ahead of Saturday's 'Rewind' of France v Argentina at World Cup 2018, BBC Sport looks back at Kylian Mbappe's sensational contribution to the round of 16 match. - https://www.bbc.co.uk/sport/av/football/52157663\"}"}}
{"HASH": {"string": "LHOFVHTKKWHB9VIXAHCOWNLMNZHWVRECBSNGJXDZONOOFMUZGAYCINSVTIXK9IBV9HNT9UTKQJBWZ9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585926603}, "PAYLOAD": {"string": "{\"channel\": \"TEST\", \"text\": \"PING PSWEFCAJONTUDMFRCDDYIUJH9KJDZEYVDPPBHM9KWWGFKRYKJ9O99PLTLUBPMNKANDPPAO9SOOSAOOSAV\"}"}}
{"HASH": {"string": "J9SDEGFATSPDLYNKLJFCHTBVLSPACNTYBOEZCLKIDIHAGNKGLBSSQ9JYZNILBNUEKJMJZMQD9SLRZ9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585926661}, "PAYLOAD": {"string": "{\"channel\": \"TWEETS\", \"text\": \"{\\\"text\\\":\\\"Introduction to Coordicide Specifications  by the #IOTA Research team  &amp; now publicly shared with the $IOTA Communi\\u2026 https://t.co/r44HuV1iDw\\\", \\\"url\\\":\\\"https://twitter.com/PeterInAsia1/status/1246090485669617665\\\"}\"}"}}
{"HASH": {"string": "WPNTSLSRPDKKMVNICEHFPTXAEGNIDHYRHMDP9PDFYETQFYQRDISBVMNCBTFSWDKXFXSKPFICHICIA9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585926668}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"South Africa's ruthlessly efficient fight against coronavirus - The government seems to have acted faster than many other states to tackle Covid-19, writes Andrew Harding. - https://www.bbc.co.uk/news/world-africa-52125713\"}"}}
{"HASH": {"string": "MPXFXOIKMRLWGPMNPCSRYEHUEYGNLUNQTKHMGZBWHQKNELKDUHVUGUGPEGXEHCVFXKBZKDDPZSDPZ9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585927203}, "PAYLOAD": {"string": "{\"channel\": \"TEST\", \"text\": \"PING BWGZYBWIUYOHNQYPYLKCXSZKSPL9TLELVNFGBCQLZVJRMXSNDGMTDGGFNEISONQBRNSLTOZSLMFZEMDZF\"}"}}
{"HASH": {"string": "ZIAKKEJMPLBWZRUXVTUJSEZWN9FMNAKULGSDDCETGATVU9QZNLXDHPPUWCJVUVMW9PKQDUNEOXTOA9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585927262}, "PAYLOAD": {"string": "{\"channel\": \"TWEETS\", \"text\": \"{\\\"text\\\":\\\"IOTA Streams goes live: Alpha version released #iota #iotastreams #miota\\nhttps://t.co/K7yKGYJSwR\\\", \\\"url\\\":\\\"https://twitter.com/CryptoNewsFlas3/status/1246094234345779201\\\"}\"}"}}
{"HASH": {"string": "VWLUPN9ADHFHPLKFIIBYHQEFQCMSHYUCNZZYNYBMZEJOEBBFUZVVGWXVAKCRXNCJHXYWDCSDOUNW99999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585927267}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"Coronavirus: Garden centres switch to virtual personal shopping - Plants are being sold via livestreams, emails and social media as lockdown forces traders to close. - https://www.bbc.co.uk/news/technology-52153475\"}"}}
{"HASH": {"string": "WQFR9IJQKXYXSGLPIELFFLFVUCKUVULHLVPFIVLFWUZJPWENTXCTJFIIWCTZRZZFVRPHGZHJOLVSA9999"}, "ADDRESS": {"string": "RCUBQQVCGYZJVLMJ9UVVQTLKHGOKANGB9SUA9ZO9QGGDHLRWNXHUQGBDCORNICKIGZWPJXU9PLX9AINAX"}, "VALUE": {"long": 0}, "TAG": {"string": "\u019E/\u01A1\f"}, "TIMESTAMP": {"long": 1585927275}, "PAYLOAD": {"string": "{\"channel\": \"NEWS\", \"text\": \"'The fire was slowly burning out' - Olympic champion Ransley on why Tokyo delay has forced retirement - British rower Tom Ransley announces his retirement after the Olympics are delayed for a year because of coronavirus. - https://www.bbc.co.uk/sport/rowing/52140705\"}"}}
```

## Next step: Time and Window queries

https://docs.confluent.io/current/ksql/docs/concepts/time-and-windows-in-ksql-queries.html
