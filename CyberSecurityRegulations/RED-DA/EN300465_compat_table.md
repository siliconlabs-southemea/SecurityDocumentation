---
sort: 2
---
# ETSI EN 303 645 Mapping to Secure Vault

| Requirement                                                                                                                                                                                                      | Secure Vault Function                                                  |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| 5.2 Implement a means to manage reports of vulnerabilities                                                                                                                                                       | silabs.com/security/product-security                                   |
| 5.3 Keep Software Updated - all software components in consumer IoT devices should be securely updatable                                                                                                         | Secure OTA w/ Gecko Bootloader                                         |
| 5.4 Securely store sensitive security parameters                                                                                                                                                                 | OTP and Secure Key Management                                          |
| 5.4-2 Where a hard-coded unique per device identity is used in a device for security purposes, it shall be implemented in such a way that it resists tampering by means such as physical, electrical or software | Secure Identity / Attestation and Anti-Tamper                          |
| 5.5.1 Th econsumer IoT device shall use best practice cryptography to communicate securely                                                                                                                       | Follow Latest crypto standards of Zigbee, Z-Wave, Thread, BLE and WiFi |
| 5.5-2 The consumer IoT device should use reviewed or evaluated implementations to deliver network and security functionalities, particularly in the field of cryptography.                                       | CAVP testing for all crypto algorithms                                 |
| 5.6 Minimize exposed attack surfaces                                                                                                                                                                             | Secure Debug                                                           |
