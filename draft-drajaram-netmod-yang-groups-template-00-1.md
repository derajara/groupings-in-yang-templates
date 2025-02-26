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

         ........

      /*
       * Identities
       */

          ........

      /*
       * Features
       */

          ........
      // Grouping for mandatory attributes
      grouping mandatory-attributes {
            leaf type {
            type identityref {
              base interface-type;
            }
     /*     mandatory true;     Removing mandatories*/
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
    /*      default "true";    Removing defaults */
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
            .......

         leaf name {
            type string;
            description
              "The name of the interface.
               .....";
          }

          leaf description {
            type string;
            description
              "A textual description of the interface.
               ......";
          }
          uses mandatory-attributes{
          refine type{
          mandatory true;
          }
          }
    /*      
            leaf type {
            type identityref {
              base interface-type;
            }
            mandatory true;
            description
              "The type of the interface.";
          }
    */
          uses default-attributes{
          refine enabled{
          default "true";
          }
         }
    /*      
            leaf enabled {
            type boolean;
            default "true";
            description
              "This leaf contains the configured, desired state of the
               interface.";
          }
    */
          leaf link-up-down-trap-enable {
               .....
          }

          leaf admin-status {
                ........
          }
          .............

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