---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###

title: "Groupings in the context of Templates."
abbrev: "yang-groups-template"
category: info

docname: draft-drajaram-netmod-yang-groups-template-00
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ops
workgroup: netmod
keyword:
 - templates
 - profiles
 - yang scalability
 - groupings
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Deepak Rajaram
    organization: Nokia
    city: Chennai
    email: deepak.rajaram@nokia.com

 -
    fullname: Robert Peschi
    organization: Nokia
    city: Antwerp
    email: robert.peschi@nokia.com
 -
    fullname: Shiya Ashraf
    organization: Nokia
    city: Antwerp
    email: shiya.ashraf@nokia.com
normative:

informative:


--- abstract

This document explores the importance of groupings in the context of templates[https://datatracker.ietf.org/doc/draft-rajaram-netmod-yang-cfg-template-framework/], their advantages, and provides practical examples to illustrate their usage. Additionally, we discuss adapting existing YANG models to use groupings effectively while ensuring structural integrity.

--- middle

# Introduction

One of the key features of YANG is the use of "groupings", which allow for modular and reusable schema components. Groupings enable better organization, efficiency, and consistency in YANG models, making them a best practice in network modeling.

# Understanding Groupings in YANG

A "groupings" on it's own do not instantiate data nodes by themselves but it is a reusable 'blueprint' that defines a set of schema nodes.It can be used within container, list, or other YANG constructs using the "uses" statement.Groupings help avoid repetition and enable the modularization of models.


# Understanding Groupings in Templates
Templates are logical structures in YANG that standardize configuration patterns across multiple template-consumers/instances. They enable network operators to create reusable configurations for devices, interfaces, or services. Templates often leverage groupings to encapsulate standard structures that can be instantiated multiple times.While groupings in general acts as a blueprint, it is extremly usefull when used with yang templates.It promotes the concept of DRY(dont repeat yourself) while allowing modification or overwriting the value of a data node in the template-consumer(instance). The resultant configuration will be a merge of the values coming from template and the values comings from the template-consumer, the values from template-consumer takes higher precedance.
  
A YANG template can be constructed using groupings to define standard configuration blocks, which are then applied to various parts of a model.

Example: Creating a Reusable Interface Template

Step 1: Define a Grouping and Instantiate It in a Template

    module interface-template {
        namespace "http://example.com/interface-template";
        prefix intf;

        grouping interface-config {
            leaf interface-name {
                type string;
            }
            leaf ip-address {
                type string;
            }
            leaf subnet-mask {
                type string;
            }
        }

        container interface-template {
            uses interface-config {
                refine ip-address {
                  default "192.168.1.1";
                }
                refine subnet-mask {
                  default "255.255.255.0";
                }
            }
        }
    }

Configuration Instance of interface-template

    <interface-template>
     <interface-name>eth0</interface-name>
     <ip-address>192.168.1.3</ip-address>
     <subnet-mask>255.255.255.0</subnet-mask>
    </interface-template>

Step 2: Override(if needed) Certain Values in a Template Consumer

    module network-config {
        namespace "http://example.com/network-config";
        prefix netcfg;

        import interface-template {
            prefix intf;
        }

        container interfaces {
            container interface-instance {
                uses intf:interface-config {
                    refine ip-address {
                        default "192.168.2.1";  
                        // Overriding configured IP from the template.
                    }
                    refine subnet-mask {
                    default "255.255.255.128";  // Overriding subnet mask
                    }
                }
            }
        }
    }

Step 3: Instantiate the Template in a Configuration Instance

    <interfaces>
    <interface-instance>
        <interface-name>eth0</interface-name>
        <ip-address>192.168.2.1</ip-address>
        <subnet-mask>255.255.255.128</subnet-mask>
    </interface-instance>
    </interfaces>

The interface-template module defines a grouping called interface-config, which contains interface-related configurations.

The interface-template container within interface-template instantiates the grouping, assigning default values for ip-address and subnet-mask.

The network-config module imports interface-template, but instead of using interface-template container, it directly applies interface-config to allow overriding specific values within interface-instance.

The instance configuration reflects these overrides while keeping the template structure consistent.



# Adapting Existing YANG Models to Use Groupings

Many existing YANG models might have redundant structures that can be optimized by introducing groupings. Below are steps to adapt an existing model:

## Identify Repetitive Structures

Look for data nodes or subtrees that appear multiple times in the model. These are ideal candidates for extraction into a grouping.

## Define a Grouping

Create a `grouping` structure containing the common elements.

## Replace Redundant Code with Uses Statement

Use the `uses` statement to replace the repeated structures in the model.

### Example: Converting an Existing Model to Use Groupings

 A conceptual Original Model Without Groupings is below, It oulines a simple use of groupings without a practical need to merge or override certain configurations which is outlined further in a more detailed standard model later. 

    module existing-model {
    namespace "http://example.com/existing-model";
    prefix existing;

    container device1 {
        leaf ip-address {
            type string;
        }
        leaf subnet-mask {
            type string;
        }
    }

    container device2 {
        leaf ip-address {
            type string;
        }
        leaf subnet-mask {
            type string;
        }
    }
    }


 Optimized Model Using Groupings


    module optimized-model {
    namespace "http://example.com/optimized-model";
    prefix opt;

    grouping ip-config {
        leaf ip-address {
            type string;
        }
        leaf subnet-mask {
            type string;
        }
    }

    container device1 {
        uses ip-config;
    }

    container device2 {
        uses ip-config;
    }
    }

Similar to the above concept, one could optimize exisiting standard model. Considering re-usability(usage of groupings) alone does not solve pragmatic usecases,other aspects like configuration merge and configuration overriding must also be considered.This is important irrespective of how a template-consumer inherits from a template. ie: a template-consumer could be a set of list elements which inhertits from a common template or it could also be a virtual device which uses schema mounted models that are same as the host device. for this example, the ietf-interfaces model is choosen and the groupings defined herein could be used both for the template as well as the template-consumer.

    module ietf-interfaces {
      yang-version 1.1;
      namespace "urn:ietf:params:xml:ns:yang:ietf-interfaces";
      prefix if;

      import ietf-yang-types {
        prefix yang;
      }

      organization
        "IETF NETMOD (Network Modeling) Working Group";

      contact
        "WG Web:   <https://datatracker.ietf.org/wg/netmod/>
         WG List:  <mailto:netmod@ietf.org>

         Editor:   Martin Bjorklund
                   <mailto:mbj@tail-f.com>";

      description
        "This module contains a collection of YANG definitions for
         managing network interfaces.

         Copyright (c) 2018 IETF Trust and the persons identified as
         authors of the code.  All rights reserved.

         Redistribution and use in source and binary forms, with or
         without modification, is permitted pursuant to, and subject
         to the license terms contained in, the Simplified BSD License
         set forth in Section 4.c of the IETF Trust's Legal Provisions
         Relating to IETF Documents
         (https://trustee.ietf.org/license-info).

         This version of this YANG module is part of RFC 8343; see
         the RFC itself for full legal notices.";

      revision 2018-02-20 {
        description
          "Updated to support NMDA.";
        reference
          "RFC 8343: A YANG Data Model for Interface Management";
      }

      revision 2014-05-08 {
        description
          "Initial revision.";
        reference
          "RFC 7223: A YANG Data Model for Interface Management";
      }

      /*
       * Typedefs
       */

      typedef interface-ref {
        type leafref {
          path "/if:interfaces/if:interface/if:name";
        }
        description
          "This type is used by data models that need to reference
           interfaces.";
      }

      /*
       * Identities
       */

      identity interface-type {
        description
          "Base identity from which specific interface types are
           derived.";
      }

      /*
       * Features
       */

      feature arbitrary-names {
        description
          "This feature indicates that the device allows user-controlled
           interfaces to be named arbitrarily.";
      }
      feature pre-provisioning {
        description
          "This feature indicates that the device supports
           pre-provisioning of interface configuration, i.e., it is
           possible to configure an interface whose physical interface
           hardware is not present on the device.";
      }
      feature if-mib {
        description
          "This feature indicates that the device implements
           the IF-MIB.";
        reference
          "RFC 2863: The Interfaces Group MIB";
      }
        // Grouping for mandatory attributes
      grouping mandatory-attributes {
            leaf type {
            type identityref {
              base interface-type;
            }
     /*       mandatory true;     Removing mandatories*/
            description
              "The type of the interface.

               When an interface entry is created, a server MAY
               initialize the type leaf with a valid value, e.g., if it
               is possible to derive the type from the name of the
               interface.

               If a client tries to set the type of an interface to a
               value that can never be used by the system, e.g., if the
               type is not supported or if the type does not match the
               name of the interface, the server MUST reject the request.
               A NETCONF server MUST reply with an rpc-error with the
               error-tag 'invalid-value' in this case.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifType";
          }
        }
        grouping default-attributes {
         leaf enabled {
            type boolean;
    /*        default "true";    Removing defaults */
            description
              "This leaf contains the configured, desired state of the
               interface.

               Systems that implement the IF-MIB use the value of this
               leaf in the intended configuration to set
               IF-MIB.ifAdminStatus to 'up' or 'down' after an ifEntry
               has been initialized, as described in RFC 2863.

               Changes in this leaf in the intended configuration are
               reflected in ifAdminStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifAdminStatus";
          }
        }
      /*
       * Data nodes
       */

      container interfaces {
        description
          "Interface parameters.";

        list interface {
          key "name";

          description
            "The list of interfaces on the device.

             The status of an interface is available in this list in the
             operational state.  If the configuration of a
             system-controlled interface cannot be used by the system
             (e.g., the interface hardware present does not match the
             interface type), then the configuration is not applied to
             the system-controlled interface shown in the operational
             state.  If the configuration of a user-controlled interface
             cannot be used by the system, the configured interface is
             not instantiated in the operational state.

             System-controlled interfaces created by the system are
             always present in this list in the operational state,
             whether or not they are configured.";

         leaf name {
            type string;
            description
              "The name of the interface.

               A device MAY restrict the allowed values for this leaf,
               possibly depending on the type of the interface.
               For system-controlled interfaces, this leaf is the
               device-specific name of the interface.

               If a client tries to create configuration for a
               system-controlled interface that is not present in the
               operational state, the server MAY reject the request if
               the implementation does not support pre-provisioning of
               interfaces or if the name refers to an interface that can
               never exist in the system.  A Network Configuration
               Protocol (NETCONF) server MUST reply with an rpc-error
               with the error-tag 'invalid-value' in this case.

               If the device supports pre-provisioning of interface
               configuration, the 'pre-provisioning' feature is
               advertised.

               If the device allows arbitrarily named user-controlled
               interfaces, the 'arbitrary-names' feature is advertised.

               When a configured user-controlled interface is created by
               the system, it is instantiated with the same name in the
               operational state.

               A server implementation MAY map this leaf to the ifName
               MIB object.  Such an implementation needs to use some
               mechanism to handle the differences in size and characters
               allowed between this leaf and ifName.  The definition of
               such a mechanism is outside the scope of this document.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifName";
          }

          leaf description {
            type string;
            description
              "A textual description of the interface.

               A server implementation MAY map this leaf to the ifAlias
               MIB object.  Such an implementation needs to use some
               mechanism to handle the differences in size and characters
               allowed between this leaf and ifAlias.  The definition of
               such a mechanism is outside the scope of this document.

               Since ifAlias is defined to be stored in non-volatile
               storage, the MIB implementation MUST map ifAlias to the
               value of 'description' in the persistently stored
               configuration.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifAlias";
          }
          uses mandatory-attributes{
          refine type{
          mandatory true;
          }
          }
    /*      leaf type {
            type identityref {
              base interface-type;
            }
            mandatory true;
            description
              "The type of the interface.

               When an interface entry is created, a server MAY
               initialize the type leaf with a valid value, e.g., if it
               is possible to derive the type from the name of the
               interface.

               If a client tries to set the type of an interface to a
               value that can never be used by the system, e.g., if the
               type is not supported or if the type does not match the
               name of the interface, the server MUST reject the request.
               A NETCONF server MUST reply with an rpc-error with the
               error-tag 'invalid-value' in this case.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifType";
          }
    */
    uses default-attributes{
    refine enabled{
    default "true";
        }
      }
    /*      leaf enabled {
            type boolean;
            default "true";
            description
              "This leaf contains the configured, desired state of the
               interface.

               Systems that implement the IF-MIB use the value of this
               leaf in the intended configuration to set
               IF-MIB.ifAdminStatus to 'up' or 'down' after an ifEntry
               has been initialized, as described in RFC 2863.

               Changes in this leaf in the intended configuration are
               reflected in ifAdminStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifAdminStatus";
          }
    */
          leaf link-up-down-trap-enable {
            if-feature if-mib;
            type enumeration {
              enum enabled {
                value 1;
                description
                  "The device will generate linkUp/linkDown SNMP
                   notifications for this interface.";
              }
              enum disabled {
                value 2;
                description
                  "The device will not generate linkUp/linkDown SNMP
                   notifications for this interface.";
              }
            }
            description
              "Controls whether linkUp/linkDown SNMP notifications
               should be generated for this interface.

               If this node is not configured, the value 'enabled' is
               operationally used by the server for interfaces that do
               not operate on top of any other interface (i.e., there are
               no 'lower-layer-if' entries), and 'disabled' otherwise.";
            reference
              "RFC 2863: The Interfaces Group MIB -
                         ifLinkUpDownTrapEnable";
          }

          leaf admin-status {
            if-feature if-mib;
            type enumeration {
              enum up {
                value 1;
                description
                  "Ready to pass packets.";
              }
              enum down {
                value 2;
                description
                  "Not ready to pass packets and not in some test mode.";
              }
              enum testing {
                value 3;
                description
                  "In some test mode.";
              }
            }
            config false;
            mandatory true;
            description
              "The desired state of the interface.

               This leaf has the same read semantics as ifAdminStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifAdminStatus";
          }

          leaf oper-status {
            type enumeration {
              enum up {
                value 1;
                description
                  "Ready to pass packets.";
              }
              enum down {
                value 2;

                description
                  "The interface does not pass any packets.";
              }
              enum testing {
                value 3;
                description
                  "In some test mode.  No operational packets can
                   be passed.";
              }
              enum unknown {
                value 4;
                description
                  "Status cannot be determined for some reason.";
              }
              enum dormant {
                value 5;
                description
                  "Waiting for some external event.";
              }
              enum not-present {
                value 6;
                description
                  "Some component (typically hardware) is missing.";
              }
              enum lower-layer-down {
                value 7;
                description
                  "Down due to state of lower-layer interface(s).";
              }
            }
            config false;
            mandatory true;
            description
              "The current operational state of the interface.

               This leaf has the same semantics as ifOperStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifOperStatus";
          }

          leaf last-change {
            type yang:date-and-time;
            config false;
            description
              "The time the interface entered its current operational
               state.  If the current state was entered prior to the
               last re-initialization of the local network management
               subsystem, then this node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifLastChange";
          }

          leaf if-index {
            if-feature if-mib;
            type int32 {
              range "1..2147483647";
            }
            config false;
            mandatory true;
            description
              "The ifIndex value for the ifEntry represented by this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifIndex";
          }

          leaf phys-address {
            type yang:phys-address;
            config false;
            description
              "The interface's address at its protocol sub-layer.  For
               example, for an 802.x interface, this object normally
               contains a Media Access Control (MAC) address.  The
               interface's media-specific modules must define the bit
               and byte ordering and the format of the value of this
               object.  For interfaces that do not have such an address
               (e.g., a serial line), this node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifPhysAddress";
          }

          leaf-list higher-layer-if {
            type interface-ref;
            config false;
            description
              "A list of references to interfaces layered on top of this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifStackTable";
          }

          leaf-list lower-layer-if {
            type interface-ref;
            config false;

            description
              "A list of references to interfaces layered underneath this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifStackTable";
          }

          leaf speed {
            type yang:gauge64;
            units "bits/second";
            config false;
            description
                "An estimate of the interface's current bandwidth in bits
                 per second.  For interfaces that do not vary in
                 bandwidth or for those where no accurate estimation can
                 be made, this node should contain the nominal bandwidth.
                 For interfaces that have no concept of bandwidth, this
                 node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB -
                         ifSpeed, ifHighSpeed";
          }

          container statistics {
            config false;
            description
              "A collection of interface-related statistics objects.";

            leaf discontinuity-time {
              type yang:date-and-time;
              mandatory true;
              description
                "The time on the most recent occasion at which any one or
                 more of this interface's counters suffered a
                 discontinuity.  If no such discontinuities have occurred
                 since the last re-initialization of the local management
                 subsystem, then this node contains the time the local
                 management subsystem re-initialized itself.";
            }

            leaf in-octets {
              type yang:counter64;
              description
                "The total number of octets received on the interface,
                 including framing characters.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCInOctets";
            }

            leaf in-unicast-pkts {
              type yang:counter64;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were not addressed to a
                 multicast or broadcast address at this sub-layer.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCInUcastPkts";
            }

            leaf in-broadcast-pkts {
              type yang:counter64;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were addressed to a broadcast
                 address at this sub-layer.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCInBroadcastPkts";
            }

            leaf in-multicast-pkts {
              type yang:counter64;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were addressed to a multicast
                 address at this sub-layer.  For a MAC-layer protocol,
                 this includes both Group and Functional addresses.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCInMulticastPkts";
            }

            leaf in-discards {
              type yang:counter32;
              description
                "The number of inbound packets that were chosen to be
                 discarded even though no errors had been detected to
                 prevent their being deliverable to a higher-layer
                 protocol.  One possible reason for discarding such a
                 packet could be to free up buffer space.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInDiscards";
            }

            leaf in-errors {
              type yang:counter32;
              description
                "For packet-oriented interfaces, the number of inbound
                 packets that contained errors preventing them from being
                 deliverable to a higher-layer protocol.  For character-
                 oriented or fixed-length interfaces, the number of
                 inbound transmission units that contained errors
                 preventing them from being deliverable to a higher-layer
                 protocol.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInErrors";
            }

            leaf in-unknown-protos {
              type yang:counter32;

              description
                "For packet-oriented interfaces, the number of packets
                 received via the interface that were discarded because
                 of an unknown or unsupported protocol.  For
                 character-oriented or fixed-length interfaces that
                 support protocol multiplexing, the number of
                 transmission units received via the interface that were
                 discarded because of an unknown or unsupported protocol.
                 For any interface that does not support protocol
                 multiplexing, this counter is not present.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInUnknownProtos";
            }

            leaf out-octets {
              type yang:counter64;
              description
                "The total number of octets transmitted out of the
                 interface, including framing characters.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCOutOctets";
            }

            leaf out-unicast-pkts {
              type yang:counter64;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were not addressed
                 to a multicast or broadcast address at this sub-layer,
                 including those that were discarded or not sent.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCOutUcastPkts";
            }

            leaf out-broadcast-pkts {
              type yang:counter64;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were addressed to a
                 broadcast address at this sub-layer, including those
                 that were discarded or not sent.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCOutBroadcastPkts";
            }

            leaf out-multicast-pkts {
              type yang:counter64;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were addressed to a
                 multicast address at this sub-layer, including those
                 that were discarded or not sent.  For a MAC-layer
                 protocol, this includes both Group and Functional
                 addresses.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCOutMulticastPkts";
            }

            leaf out-discards {
              type yang:counter32;
              description
                "The number of outbound packets that were chosen to be
                 discarded even though no errors had been detected to
                 prevent their being transmitted.  One possible reason
                 for discarding such a packet could be to free up buffer
                 space.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifOutDiscards";
            }

            leaf out-errors {
              type yang:counter32;
              description
                "For packet-oriented interfaces, the number of outbound
                 packets that could not be transmitted because of errors.
                 For character-oriented or fixed-length interfaces, the
                 number of outbound transmission units that could not be
                 transmitted because of errors.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifOutErrors";
            }
          }

        }
      }

      /*
       * Legacy typedefs
       */

      typedef interface-state-ref {
        type leafref {
          path "/if:interfaces-state/if:interface/if:name";
        }
        status deprecated;
        description
          "This type is used by data models that need to reference
           the operationally present interfaces.";
      }

      /*
       * Legacy operational state data nodes
       */

      container interfaces-state {
        config false;
        status deprecated;
        description
          "Data nodes for the operational state of interfaces.";

        list interface {
          key "name";
          status deprecated;

          description
            "The list of interfaces on the device.

             System-controlled interfaces created by the system are
             always present in this list, whether or not they are
             configured.";

          leaf name {
            type string;
            status deprecated;
            description
              "The name of the interface.

               A server implementation MAY map this leaf to the ifName
               MIB object.  Such an implementation needs to use some
               mechanism to handle the differences in size and characters
               allowed between this leaf and ifName.  The definition of
               such a mechanism is outside the scope of this document.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifName";
          }

          leaf type {
            type identityref {
              base interface-type;
            }
            mandatory true;
            status deprecated;
            description
              "The type of the interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifType";
          }

          leaf admin-status {
            if-feature if-mib;
            type enumeration {
              enum up {
                value 1;
                description
                  "Ready to pass packets.";
              }
              enum down {
                value 2;
                description
                  "Not ready to pass packets and not in some test mode.";
              }
              enum testing {
                value 3;
                description
                  "In some test mode.";
              }
            }
            mandatory true;
            status deprecated;
            description
              "The desired state of the interface.

               This leaf has the same read semantics as ifAdminStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifAdminStatus";
          }

          leaf oper-status {
            type enumeration {
              enum up {
                value 1;
                description
                  "Ready to pass packets.";
              }
              enum down {
                value 2;
                description
                  "The interface does not pass any packets.";
              }
              enum testing {
                value 3;
                description
                  "In some test mode.  No operational packets can
                   be passed.";
              }
              enum unknown {
                value 4;
                description
                  "Status cannot be determined for some reason.";
              }
              enum dormant {
                value 5;
                description
                  "Waiting for some external event.";
              }
              enum not-present {
                value 6;
                description
                  "Some component (typically hardware) is missing.";
              }
              enum lower-layer-down {
                value 7;
                description
                  "Down due to state of lower-layer interface(s).";
              }
            }
            mandatory true;
            status deprecated;
            description
              "The current operational state of the interface.

               This leaf has the same semantics as ifOperStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifOperStatus";
          }

          leaf last-change {
            type yang:date-and-time;
            status deprecated;
            description
              "The time the interface entered its current operational
               state.  If the current state was entered prior to the
               last re-initialization of the local network management
               subsystem, then this node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifLastChange";
          }

          leaf if-index {
            if-feature if-mib;
            type int32 {
              range "1..2147483647";
            }
            mandatory true;
            status deprecated;
            description
              "The ifIndex value for the ifEntry represented by this
               interface.";

            reference
              "RFC 2863: The Interfaces Group MIB - ifIndex";
          }

          leaf phys-address {
            type yang:phys-address;
            status deprecated;
            description
              "The interface's address at its protocol sub-layer.  For
               example, for an 802.x interface, this object normally
               contains a Media Access Control (MAC) address.  The
               interface's media-specific modules must define the bit
               and byte ordering and the format of the value of this
               object.  For interfaces that do not have such an address
               (e.g., a serial line), this node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifPhysAddress";
          }

          leaf-list higher-layer-if {
            type interface-state-ref;
            status deprecated;
            description
              "A list of references to interfaces layered on top of this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifStackTable";
          }

          leaf-list lower-layer-if {
            type interface-state-ref;
            status deprecated;
            description
              "A list of references to interfaces layered underneath this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifStackTable";
          }

          leaf speed {
            type yang:gauge64;
            units "bits/second";
            status deprecated;
            description
                "An estimate of the interface's current bandwidth in bits
                 per second.  For interfaces that do not vary in
                 bandwidth or for those where no accurate estimation can

                 be made, this node should contain the nominal bandwidth.
                 For interfaces that have no concept of bandwidth, this
                 node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB -
                         ifSpeed, ifHighSpeed";
          }

          container statistics {
            status deprecated;
            description
              "A collection of interface-related statistics objects.";

            leaf discontinuity-time {
              type yang:date-and-time;
              mandatory true;
              status deprecated;
              description
                "The time on the most recent occasion at which any one or
                 more of this interface's counters suffered a
                 discontinuity.  If no such discontinuities have occurred
                 since the last re-initialization of the local management
                 subsystem, then this node contains the time the local
                 management subsystem re-initialized itself.";
            }

            leaf in-octets {
              type yang:counter64;
              status deprecated;
              description
                "The total number of octets received on the interface,
                 including framing characters.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCInOctets";
            }

            leaf in-unicast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were not addressed to a
                 multicast or broadcast address at this sub-layer.
                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCInUcastPkts";
            }

            leaf in-broadcast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were addressed to a broadcast
                 address at this sub-layer.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCInBroadcastPkts";
            }

            leaf in-multicast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were addressed to a multicast
                 address at this sub-layer.  For a MAC-layer protocol,
                 this includes both Group and Functional addresses.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCInMulticastPkts";
            }

            leaf in-discards {
              type yang:counter32;
              status deprecated;

              description
                "The number of inbound packets that were chosen to be
                 discarded even though no errors had been detected to
                 prevent their being deliverable to a higher-layer
                 protocol.  One possible reason for discarding such a
                 packet could be to free up buffer space.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInDiscards";
            }

            leaf in-errors {
              type yang:counter32;
              status deprecated;
              description
                "For packet-oriented interfaces, the number of inbound
                 packets that contained errors preventing them from being
                 deliverable to a higher-layer protocol.  For character-
                 oriented or fixed-length interfaces, the number of
                 inbound transmission units that contained errors
                 preventing them from being deliverable to a higher-layer
                 protocol.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInErrors";
            }

            leaf in-unknown-protos {
              type yang:counter32;
              status deprecated;
              description
                "For packet-oriented interfaces, the number of packets
                 received via the interface that were discarded because
                 of an unknown or unsupported protocol.  For
                 character-oriented or fixed-length interfaces that
                 support protocol multiplexing, the number of
                 transmission units received via the interface that were
                 discarded because of an unknown or unsupported protocol.
                 For any interface that does not support protocol
                 multiplexing, this counter is not present.
                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInUnknownProtos";
            }

            leaf out-octets {
              type yang:counter64;
              status deprecated;
              description
                "The total number of octets transmitted out of the
                 interface, including framing characters.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCOutOctets";
            }

            leaf out-unicast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were not addressed
                 to a multicast or broadcast address at this sub-layer,
                 including those that were discarded or not sent.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCOutUcastPkts";
            }

            leaf out-broadcast-pkts {
              type yang:counter64;
              status deprecated;

              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were addressed to a
                 broadcast address at this sub-layer, including those
                 that were discarded or not sent.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCOutBroadcastPkts";
            }

            leaf out-multicast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were addressed to a
                 multicast address at this sub-layer, including those
                 that were discarded or not sent.  For a MAC-layer
                 protocol, this includes both Group and Functional
                 addresses.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCOutMulticastPkts";
            }

            leaf out-discards {
              type yang:counter32;
              status deprecated;
              description
                "The number of outbound packets that were chosen to be
                 discarded even though no errors had been detected to
                 prevent their being transmitted.  One possible reason
                 for discarding such a packet could be to free up buffer
                 space.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifOutDiscards";
            }

            leaf out-errors {
              type yang:counter32;
              status deprecated;
              description
                "For packet-oriented interfaces, the number of outbound
                 packets that could not be transmitted because of errors.
                 For character-oriented or fixed-length interfaces, the
                 number of outbound transmission units that could not be
                 transmitted because of errors.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifOutErrors";
            }
          }
        }
      }
    }

### Example: Converting an Existing Model to Use Groupings with feature flag
Another alternative is to use "if-feature" in tandem with "choice" statement to mutually exclude the existing mandatory and default nodes when template solution is used. this method ensures that existing model's functionality is not impacted whereas if a implementation chooses to include template solution, then use the new nodes which is defined in a grouping without mandatory and defaults. As an example the ietf-interfaces is considered with the necessary updates.

    module ietf-interfaces {
      yang-version 1.1;
      namespace "urn:ietf:params:xml:ns:yang:ietf-interfaces";
      prefix if;

      import ietf-yang-types {
        prefix yang;
      }

      organization
        "IETF NETMOD (Network Modeling) Working Group";

      contact
        "WG Web:   <https://datatracker.ietf.org/wg/netmod/>
         WG List:  <mailto:netmod@ietf.org>

         Editor:   Martin Bjorklund
                   <mailto:mbj@tail-f.com>";

      description
        "This module contains a collection of YANG definitions for
         managing network interfaces.

         Copyright (c) 2018 IETF Trust and the persons identified as
         authors of the code.  All rights reserved.

         Redistribution and use in source and binary forms, with or
         without modification, is permitted pursuant to, and subject
         to the license terms contained in, the Simplified BSD License
         set forth in Section 4.c of the IETF Trust's Legal Provisions
         Relating to IETF Documents
         (https://trustee.ietf.org/license-info).

         This version of this YANG module is part of RFC 8343; see
         the RFC itself for full legal notices.";

      revision 2018-02-20 {
        description
          "Updated to support NMDA.";
        reference
          "RFC 8343: A YANG Data Model for Interface Management";
      }

      revision 2014-05-08 {
        description
          "Initial revision.";
        reference
          "RFC 7223: A YANG Data Model for Interface Management";
      }

      /*
       * Groupings
       */
    grouping interface-enabled {
        description
          "Interface config true and mandatory/default data nodes.";
        leaf itf-enabled {
              type boolean;
              /*default "true";*/
              description
                "This leaf contains the configured, desired state of the
                 interface.

                 Systems that implement the IF-MIB use the value of this
                 leaf in the 'running' datastore to set
                 IF-MIB.ifAdminStatus to 'up' or 'down' after an ifEntry
                 has been initialized, as described in RFC 2863.

                 Changes in this leaf in the 'running' datastore are
                 reflected in ifAdminStatus, but if ifAdminStatus is
                 changed over SNMP, this leaf is not affected.";
              reference
                "RFC 2863 - ifAdminStatus";
            }
        }
    grouping interface-type{
          leaf itf-type {
            type identityref {
              base interface-type;
            }
            /*mandatory true;*/
            description
              "The type of the interface.

               When an interface entry is created, a server MAY
               initialize the type leaf with a valid value, e.g., if it
               is possible to derive the type from the name of the
               interface.

               If a client tries to set the type of an interface to a
               value that can never be used by the system, e.g., if the
               type is not supported or if the type does not match the
               name of the interface, the server MUST reject the request.
               A NETCONF server MUST reply with an rpc-error with the
               error-tag 'invalid-value' in this case.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifType";
          }
    }
      /*
       * Typedefs
       */

      typedef interface-ref {
        type leafref {
          path "/if:interfaces/if:interface/if:name";
        }
        description
          "This type is used by data models that need to reference
           interfaces.";
      }

      /*
       * Identities
       */

      identity interface-type {
        description
          "Base identity from which specific interface types are
           derived.";
      }

      /*
       * Features
       */

      feature arbitrary-names {
        description
          "This feature indicates that the device allows user-controlled
           interfaces to be named arbitrarily.";
      }
      feature pre-provisioning {
        description
          "This feature indicates that the device supports
           pre-provisioning of interface configuration, i.e., it is
           possible to configure an interface whose physical interface
           hardware is not present on the device.";
      }
      feature if-mib {
        description
          "This feature indicates that the device implements
           the IF-MIB.";
        reference
          "RFC 2863: The Interfaces Group MIB";
      }
      feature interfaces-template-groupings {
        description
          "This feature indicates that the device implements
           the groupings for templates.";
      }

      /*
       * Data nodes
       */

      container interfaces {
        description
          "Interface parameters.";

        list interface {
          key "name";

          description
            "The list of interfaces on the device.

             The status of an interface is available in this list in the
             operational state.  If the configuration of a
             system-controlled interface cannot be used by the system
             (e.g., the interface hardware present does not match the
             interface type), then the configuration is not applied to
             the system-controlled interface shown in the operational
             state.  If the configuration of a user-controlled interface
             cannot be used by the system, the configured interface is
             not instantiated in the operational state.

             System-controlled interfaces created by the system are
             always present in this list in the operational state,
             whether or not they are configured.";

         leaf name {
            type string;
            description
              "The name of the interface.

               A device MAY restrict the allowed values for this leaf,
               possibly depending on the type of the interface.
               For system-controlled interfaces, this leaf is the
               device-specific name of the interface.

               If a client tries to create configuration for a
               system-controlled interface that is not present in the
               operational state, the server MAY reject the request if
               the implementation does not support pre-provisioning of
               interfaces or if the name refers to an interface that can
               never exist in the system.  A Network Configuration
               Protocol (NETCONF) server MUST reply with an rpc-error
               with the error-tag 'invalid-value' in this case.

               If the device supports pre-provisioning of interface
               configuration, the 'pre-provisioning' feature is
               advertised.

               If the device allows arbitrarily named user-controlled
               interfaces, the 'arbitrary-names' feature is advertised.

               When a configured user-controlled interface is created by
               the system, it is instantiated with the same name in the
               operational state.

               A server implementation MAY map this leaf to the ifName
               MIB object.  Such an implementation needs to use some
               mechanism to handle the differences in size and characters
               allowed between this leaf and ifName.  The definition of
               such a mechanism is outside the scope of this document.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifName";
          }

          leaf description {
            type string;
            description
              "A textual description of the interface.

               A server implementation MAY map this leaf to the ifAlias
               MIB object.  Such an implementation needs to use some
               mechanism to handle the differences in size and characters
               allowed between this leaf and ifAlias.  The definition of
               such a mechanism is outside the scope of this document.

               Since ifAlias is defined to be stored in non-volatile
               storage, the MIB implementation MUST map ifAlias to the
               value of 'description' in the persistently stored
               configuration.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifAlias";
          }
          choice interface-type{
          case template{
          uses interface-type{ 
          if-feature interfaces-template-groupings;
          }
          }
          case non-template{
          leaf type {
            type identityref {
              base interface-type;
            }
            mandatory true;
            description
              "The type of the interface.

               When an interface entry is created, a server MAY
               initialize the type leaf with a valid value, e.g., if it
               is possible to derive the type from the name of the
               interface.

               If a client tries to set the type of an interface to a
               value that can never be used by the system, e.g., if the
               type is not supported or if the type does not match the
               name of the interface, the server MUST reject the request.
               A NETCONF server MUST reply with an rpc-error with the
               error-tag 'invalid-value' in this case.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifType";
          }
          }
    }
          choice interface-enabled{
          case template{
          uses interface-enabled{ 
          if-feature interfaces-template-groupings;
          }
          }
          case non-template{
          leaf enabled {
            type boolean;
            default "true";
            description
              "This leaf contains the configured, desired state of the
               interface.

               Systems that implement the IF-MIB use the value of this
               leaf in the intended configuration to set
               IF-MIB.ifAdminStatus to 'up' or 'down' after an ifEntry
               has been initialized, as described in RFC 2863.

               Changes in this leaf in the intended configuration are
               reflected in ifAdminStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifAdminStatus";
          }
    }
    }

          leaf link-up-down-trap-enable {
            if-feature if-mib;
            type enumeration {
              enum enabled {
                value 1;
                description
                  "The device will generate linkUp/linkDown SNMP
                   notifications for this interface.";
              }
              enum disabled {
                value 2;
                description
                  "The device will not generate linkUp/linkDown SNMP
                   notifications for this interface.";
              }
            }
            description
              "Controls whether linkUp/linkDown SNMP notifications
               should be generated for this interface.

               If this node is not configured, the value 'enabled' is
               operationally used by the server for interfaces that do
               not operate on top of any other interface (i.e., there are
               no 'lower-layer-if' entries), and 'disabled' otherwise.";
            reference
              "RFC 2863: The Interfaces Group MIB -
                         ifLinkUpDownTrapEnable";
          }

          leaf admin-status {
            if-feature if-mib;
            type enumeration {
              enum up {
                value 1;
                description
                  "Ready to pass packets.";
              }
              enum down {
                value 2;
                description
                  "Not ready to pass packets and not in some test mode.";
              }
              enum testing {
                value 3;
                description
                  "In some test mode.";
              }
            }
            config false;
            mandatory true;
            description
              "The desired state of the interface.

               This leaf has the same read semantics as ifAdminStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifAdminStatus";
          }

          leaf oper-status {
            type enumeration {
              enum up {
                value 1;
                description
                  "Ready to pass packets.";
              }
              enum down {
                value 2;

                description
                  "The interface does not pass any packets.";
              }
              enum testing {
                value 3;
                description
                  "In some test mode.  No operational packets can
                   be passed.";
              }
              enum unknown {
                value 4;
                description
                  "Status cannot be determined for some reason.";
              }
              enum dormant {
                value 5;
                description
                  "Waiting for some external event.";
              }
              enum not-present {
                value 6;
                description
                  "Some component (typically hardware) is missing.";
              }
              enum lower-layer-down {
                value 7;
                description
                  "Down due to state of lower-layer interface(s).";
              }
            }
            config false;
            mandatory true;
            description
              "The current operational state of the interface.

               This leaf has the same semantics as ifOperStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifOperStatus";
          }

          leaf last-change {
            type yang:date-and-time;
            config false;
            description
              "The time the interface entered its current operational
               state.  If the current state was entered prior to the
               last re-initialization of the local network management
               subsystem, then this node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifLastChange";
          }

          leaf if-index {
            if-feature if-mib;
            type int32 {
              range "1..2147483647";
            }
            config false;
            mandatory true;
            description
              "The ifIndex value for the ifEntry represented by this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifIndex";
          }

          leaf phys-address {
            type yang:phys-address;
            config false;
            description
              "The interface's address at its protocol sub-layer.  For
               example, for an 802.x interface, this object normally
               contains a Media Access Control (MAC) address.  The
               interface's media-specific modules must define the bit
               and byte ordering and the format of the value of this
               object.  For interfaces that do not have such an address
               (e.g., a serial line), this node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifPhysAddress";
          }

          leaf-list higher-layer-if {
            type interface-ref;
            config false;
            description
              "A list of references to interfaces layered on top of this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifStackTable";
          }

          leaf-list lower-layer-if {
            type interface-ref;
            config false;

            description
              "A list of references to interfaces layered underneath this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifStackTable";
          }

          leaf speed {
            type yang:gauge64;
            units "bits/second";
            config false;
            description
                "An estimate of the interface's current bandwidth in bits
                 per second.  For interfaces that do not vary in
                 bandwidth or for those where no accurate estimation can
                 be made, this node should contain the nominal bandwidth.
                 For interfaces that have no concept of bandwidth, this
                 node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB -
                         ifSpeed, ifHighSpeed";
          }

          container statistics {
            config false;
            description
              "A collection of interface-related statistics objects.";

            leaf discontinuity-time {
              type yang:date-and-time;
              mandatory true;
              description
                "The time on the most recent occasion at which any one or
                 more of this interface's counters suffered a
                 discontinuity.  If no such discontinuities have occurred
                 since the last re-initialization of the local management
                 subsystem, then this node contains the time the local
                 management subsystem re-initialized itself.";
            }

            leaf in-octets {
              type yang:counter64;
              description
                "The total number of octets received on the interface,
                 including framing characters.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCInOctets";
            }

            leaf in-unicast-pkts {
              type yang:counter64;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were not addressed to a
                 multicast or broadcast address at this sub-layer.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCInUcastPkts";
            }

            leaf in-broadcast-pkts {
              type yang:counter64;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were addressed to a broadcast
                 address at this sub-layer.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCInBroadcastPkts";
            }

            leaf in-multicast-pkts {
              type yang:counter64;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were addressed to a multicast
                 address at this sub-layer.  For a MAC-layer protocol,
                 this includes both Group and Functional addresses.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCInMulticastPkts";
            }

            leaf in-discards {
              type yang:counter32;
              description
                "The number of inbound packets that were chosen to be
                 discarded even though no errors had been detected to
                 prevent their being deliverable to a higher-layer
                 protocol.  One possible reason for discarding such a
                 packet could be to free up buffer space.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInDiscards";
            }

            leaf in-errors {
              type yang:counter32;
              description
                "For packet-oriented interfaces, the number of inbound
                 packets that contained errors preventing them from being
                 deliverable to a higher-layer protocol.  For character-
                 oriented or fixed-length interfaces, the number of
                 inbound transmission units that contained errors
                 preventing them from being deliverable to a higher-layer
                 protocol.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInErrors";
            }

            leaf in-unknown-protos {
              type yang:counter32;

              description
                "For packet-oriented interfaces, the number of packets
                 received via the interface that were discarded because
                 of an unknown or unsupported protocol.  For
                 character-oriented or fixed-length interfaces that
                 support protocol multiplexing, the number of
                 transmission units received via the interface that were
                 discarded because of an unknown or unsupported protocol.
                 For any interface that does not support protocol
                 multiplexing, this counter is not present.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInUnknownProtos";
            }

            leaf out-octets {
              type yang:counter64;
              description
                "The total number of octets transmitted out of the
                 interface, including framing characters.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCOutOctets";
            }

            leaf out-unicast-pkts {
              type yang:counter64;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were not addressed
                 to a multicast or broadcast address at this sub-layer,
                 including those that were discarded or not sent.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCOutUcastPkts";
            }

            leaf out-broadcast-pkts {
              type yang:counter64;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were addressed to a
                 broadcast address at this sub-layer, including those
                 that were discarded or not sent.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCOutBroadcastPkts";
            }

            leaf out-multicast-pkts {
              type yang:counter64;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were addressed to a
                 multicast address at this sub-layer, including those
                 that were discarded or not sent.  For a MAC-layer
                 protocol, this includes both Group and Functional
                 addresses.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCOutMulticastPkts";
            }

            leaf out-discards {
              type yang:counter32;
              description
                "The number of outbound packets that were chosen to be
                 discarded even though no errors had been detected to
                 prevent their being transmitted.  One possible reason
                 for discarding such a packet could be to free up buffer
                 space.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifOutDiscards";
            }

            leaf out-errors {
              type yang:counter32;
              description
                "For packet-oriented interfaces, the number of outbound
                 packets that could not be transmitted because of errors.
                 For character-oriented or fixed-length interfaces, the
                 number of outbound transmission units that could not be
                 transmitted because of errors.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifOutErrors";
            }
          }

        }
      }

      /*
       * Legacy typedefs
       */

      typedef interface-state-ref {
        type leafref {
          path "/if:interfaces-state/if:interface/if:name";
        }
        status deprecated;
        description
          "This type is used by data models that need to reference
           the operationally present interfaces.";
      }

      /*
       * Legacy operational state data nodes
       */

      container interfaces-state {
        config false;
        status deprecated;
        description
          "Data nodes for the operational state of interfaces.";

        list interface {
          key "name";
          status deprecated;

          description
            "The list of interfaces on the device.

             System-controlled interfaces created by the system are
             always present in this list, whether or not they are
             configured.";

          leaf name {
            type string;
            status deprecated;
            description
              "The name of the interface.

               A server implementation MAY map this leaf to the ifName
               MIB object.  Such an implementation needs to use some
               mechanism to handle the differences in size and characters
               allowed between this leaf and ifName.  The definition of
               such a mechanism is outside the scope of this document.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifName";
          }

          leaf type {
            type identityref {
              base interface-type;
            }
            mandatory true;
            status deprecated;
            description
              "The type of the interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifType";
          }

          leaf admin-status {
            if-feature if-mib;
            type enumeration {
              enum up {
                value 1;
                description
                  "Ready to pass packets.";
              }
              enum down {
                value 2;
                description
                  "Not ready to pass packets and not in some test mode.";
              }
              enum testing {
                value 3;
                description
                  "In some test mode.";
              }
            }
            mandatory true;
            status deprecated;
            description
              "The desired state of the interface.

               This leaf has the same read semantics as ifAdminStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifAdminStatus";
          }

          leaf oper-status {
            type enumeration {
              enum up {
                value 1;
                description
                  "Ready to pass packets.";
              }
              enum down {
                value 2;
                description
                  "The interface does not pass any packets.";
              }
              enum testing {
                value 3;
                description
                  "In some test mode.  No operational packets can
                   be passed.";
              }
              enum unknown {
                value 4;
                description
                  "Status cannot be determined for some reason.";
              }
              enum dormant {
                value 5;
                description
                  "Waiting for some external event.";
              }
              enum not-present {
                value 6;
                description
                  "Some component (typically hardware) is missing.";
              }
              enum lower-layer-down {
                value 7;
                description
                  "Down due to state of lower-layer interface(s).";
              }
            }
            mandatory true;
            status deprecated;
            description
              "The current operational state of the interface.

               This leaf has the same semantics as ifOperStatus.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifOperStatus";
          }

          leaf last-change {
            type yang:date-and-time;
            status deprecated;
            description
              "The time the interface entered its current operational
               state.  If the current state was entered prior to the
               last re-initialization of the local network management
               subsystem, then this node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifLastChange";
          }

          leaf if-index {
            if-feature if-mib;
            type int32 {
              range "1..2147483647";
            }
            mandatory true;
            status deprecated;
            description
              "The ifIndex value for the ifEntry represented by this
               interface.";

            reference
              "RFC 2863: The Interfaces Group MIB - ifIndex";
          }

          leaf phys-address {
            type yang:phys-address;
            status deprecated;
            description
              "The interface's address at its protocol sub-layer.  For
               example, for an 802.x interface, this object normally
               contains a Media Access Control (MAC) address.  The
               interface's media-specific modules must define the bit
               and byte ordering and the format of the value of this
               object.  For interfaces that do not have such an address
               (e.g., a serial line), this node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifPhysAddress";
          }

          leaf-list higher-layer-if {
            type interface-state-ref;
            status deprecated;
            description
              "A list of references to interfaces layered on top of this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifStackTable";
          }

          leaf-list lower-layer-if {
            type interface-state-ref;
            status deprecated;
            description
              "A list of references to interfaces layered underneath this
               interface.";
            reference
              "RFC 2863: The Interfaces Group MIB - ifStackTable";
          }

          leaf speed {
            type yang:gauge64;
            units "bits/second";
            status deprecated;
            description
                "An estimate of the interface's current bandwidth in bits
                 per second.  For interfaces that do not vary in
                 bandwidth or for those where no accurate estimation can

                 be made, this node should contain the nominal bandwidth.
                 For interfaces that have no concept of bandwidth, this
                 node is not present.";
            reference
              "RFC 2863: The Interfaces Group MIB -
                         ifSpeed, ifHighSpeed";
          }

          container statistics {
            status deprecated;
            description
              "A collection of interface-related statistics objects.";

            leaf discontinuity-time {
              type yang:date-and-time;
              mandatory true;
              status deprecated;
              description
                "The time on the most recent occasion at which any one or
                 more of this interface's counters suffered a
                 discontinuity.  If no such discontinuities have occurred
                 since the last re-initialization of the local management
                 subsystem, then this node contains the time the local
                 management subsystem re-initialized itself.";
            }

            leaf in-octets {
              type yang:counter64;
              status deprecated;
              description
                "The total number of octets received on the interface,
                 including framing characters.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCInOctets";
            }

            leaf in-unicast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were not addressed to a
                 multicast or broadcast address at this sub-layer.
                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCInUcastPkts";
            }

            leaf in-broadcast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were addressed to a broadcast
                 address at this sub-layer.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCInBroadcastPkts";
            }

            leaf in-multicast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The number of packets, delivered by this sub-layer to a
                 higher (sub-)layer, that were addressed to a multicast
                 address at this sub-layer.  For a MAC-layer protocol,
                 this includes both Group and Functional addresses.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCInMulticastPkts";
            }

            leaf in-discards {
              type yang:counter32;
              status deprecated;

              description
                "The number of inbound packets that were chosen to be
                 discarded even though no errors had been detected to
                 prevent their being deliverable to a higher-layer
                 protocol.  One possible reason for discarding such a
                 packet could be to free up buffer space.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInDiscards";
            }

            leaf in-errors {
              type yang:counter32;
              status deprecated;
              description
                "For packet-oriented interfaces, the number of inbound
                 packets that contained errors preventing them from being
                 deliverable to a higher-layer protocol.  For character-
                 oriented or fixed-length interfaces, the number of
                 inbound transmission units that contained errors
                 preventing them from being deliverable to a higher-layer
                 protocol.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInErrors";
            }

            leaf in-unknown-protos {
              type yang:counter32;
              status deprecated;
              description
                "For packet-oriented interfaces, the number of packets
                 received via the interface that were discarded because
                 of an unknown or unsupported protocol.  For
                 character-oriented or fixed-length interfaces that
                 support protocol multiplexing, the number of
                 transmission units received via the interface that were
                 discarded because of an unknown or unsupported protocol.
                 For any interface that does not support protocol
                 multiplexing, this counter is not present.
                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifInUnknownProtos";
            }

            leaf out-octets {
              type yang:counter64;
              status deprecated;
              description
                "The total number of octets transmitted out of the
                 interface, including framing characters.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCOutOctets";
            }

            leaf out-unicast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were not addressed
                 to a multicast or broadcast address at this sub-layer,
                 including those that were discarded or not sent.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifHCOutUcastPkts";
            }

            leaf out-broadcast-pkts {
              type yang:counter64;
              status deprecated;

              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were addressed to a
                 broadcast address at this sub-layer, including those
                 that were discarded or not sent.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCOutBroadcastPkts";
            }

            leaf out-multicast-pkts {
              type yang:counter64;
              status deprecated;
              description
                "The total number of packets that higher-level protocols
                 requested be transmitted and that were addressed to a
                 multicast address at this sub-layer, including those
                 that were discarded or not sent.  For a MAC-layer
                 protocol, this includes both Group and Functional
                 addresses.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB -
                           ifHCOutMulticastPkts";
            }

            leaf out-discards {
              type yang:counter32;
              status deprecated;
              description
                "The number of outbound packets that were chosen to be
                 discarded even though no errors had been detected to
                 prevent their being transmitted.  One possible reason
                 for discarding such a packet could be to free up buffer
                 space.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifOutDiscards";
            }

            leaf out-errors {
              type yang:counter32;
              status deprecated;
              description
                "For packet-oriented interfaces, the number of outbound
                 packets that could not be transmitted because of errors.
                 For character-oriented or fixed-length interfaces, the
                 number of outbound transmission units that could not be
                 transmitted because of errors.

                 Discontinuities in the value of this counter can occur
                 at re-initialization of the management system and at
                 other times as indicated by the value of
                 'discontinuity-time'.";
              reference
                "RFC 2863: The Interfaces Group MIB - ifOutErrors";
            }
          }
        }
      }
    }



## Advantages of Using Groupings

### Code Reusability

Groupings allow the reuse of common structures across multiple parts of a model, reducing redundancy and improving maintainability.

### Consistency

By defining structures once and reusing them, models remain consistent, preventing discrepancies between similar definitions.

### Modularity

Breaking down YANG models into reusable components improves modularity and facilitates model extension.

### Simplified Maintenance

With groupings, any necessary changes to a repeated structure only need to be made in one place, simplifying updates and maintenance.


## Conclusion

Groupings in YANG play a crucial role in implementing templates by enabling reusable, modular, and scalable configurations. By leveraging groupings, network engineers can create structured templates that enhance maintainability and ensure consistency across network configurations. Templates based on groupings simplify model updates, reduce redundancy, and promote best practices in network automation and ensure scalability for evolving network management needs.

As of today, we do not have any construct which could create a grouping "after-the-fact" since the tree is already defined. so to limit the impact to tools, we could minimise the groupings only to those nodes which is important for the correct behaviour of templates, eg: CT nodes which contains mandatory/defaults.

By adapting existing models to use groupings and integrating it effectively within templates, developers and organisations can improve operational efficiency and streamline configuration management across complex network infrastructures.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.