CVEs Disclosures

[span_0](start_span)[span_1](start_span)This document outlines the security vulnerabilities discovered in the [eProsima Micro-XRCE-DDS-Agent](https://github.com/eProsima/Micro-XRCE-DDS-Agent) during our fuzzing campaign using AFLNet[span_0](end_span)[span_1](end_span). 

[span_2](start_span)[span_3](start_span)The vulnerabilities were discovered by Jacopo Viganò and Lorenzo Della Matera and have been acknowledged by the vendor[span_2](end_span)[span_3](end_span).

---

## CVE-2025-63547: Denial of Service via Zero-length MTU Allocation

* **Vulnerability Type:** Unsafe Packet Validation (Denial of Service)
* **Affected Product:** Eprosima Micro-XRCE-DDS-Agent v.3.0.1
* **Affected Component:** Client creation inside the agent (Session object allocation)
* **Attack Type:** Remote

### Description
An issue in Eprosima Micro-XRCE-DDS Agent v.3.0.1 allows a remote attacker to cause a denial of service via a crafted packet modifying the MTU length field. 

[span_4](start_span)When a new XRCE client connects to the agent, it sends a `CREATE_CLIENT` message with a specific MTU length as one of the parameters[span_4](end_span). [span_5](start_span)The agent uses this MTU length to determine the maximum size of the buffer allocated for future responses[span_5](end_span). [span_6](start_span)If a malicious or malfunctioning client sets the MTU length to `0`, the agent attempts to allocate a buffer of size `0`, which results in a crash[span_6](end_span). 

### Attack Vector
[span_7](start_span)The Agent must receive a specially crafted `CREATE_CLIENT` packet bearing a size of `0` in the MTU length field, followed by a subsequent message (such as a `HEARTBEAT`), triggering the invalid allocation[span_7](end_span).

### References & Mitigation
* **Reported Issue:** [eProsima/Micro-XRCE-DDS-Agent#390](https://github.com/eProsima/Micro-XRCE-DDS-Agent/issues/390)
* **Status:** Acknowledged and patched by the vendor. [span_8](start_span)The fix involved patching the creation of a client representation and adding an error log in the agent CLI if a `CREATE_CLIENT` message contains an MTU of 0[span_8](end_span).

---

## CVE-2025-63548: Denial of Service via Improper Boolean Deserialization

* **Vulnerability Type:** Improper Object Deserialization (Denial of Service)
* **Affected Product:** Eprosima Micro-XRCE-DDS-Agent v.3.0.1
* **Affected Component:** Packet receiver (`InputMessage::deserialize()`)
* **Attack Type:** Remote

### Description
An issue in Eprosima Micro-XRCE-DDS Agent v.3.0.1 allows a remote attacker to cause a denial of service via a packet specially crafted to bear a non-valid value in any Boolean field.

[span_9](start_span)The Micro-XRCE-DDS Agent relies on the FastCdr library to serialize and deserialize incoming data payloads[span_9](end_span). [span_10](start_span)[span_11](start_span)During the deserialization of a boolean field, if the value is strictly different from `0` or `1`, the FastCdr library throws a `BadParamException`[span_10](end_span)[span_11](end_span). [span_12](start_span)Because this exception was previously uncaught by the caller, the agent crashes immediately upon processing the packet[span_12](end_span).

### Attack Vector
[span_13](start_span)The Agent must receive a packet specially crafted to bear a non-valid value (e.g., `0xFF`) in any boolean field (such as `m_properties_flag` in a `CREATE_CLIENT` message)[span_13](end_span). 

### References & Mitigation
* **Reported Issue:** [eProsima/Micro-XRCE-DDS-Agent#389](https://github.com/eProsima/Micro-XRCE-DDS-Agent/issues/389)
* **Status:** Acknowledged and patched by the vendor. [span_14](start_span)The issue was resolved by catching the `BadParamException` everywhere a boolean is included in a submessage, preventing the crash[span_14](end_span).
