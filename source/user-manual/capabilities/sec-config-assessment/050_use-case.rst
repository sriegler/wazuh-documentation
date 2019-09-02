.. Copyright (C) 2019 Wazuh, Inc.

Use case: Getting an alert when a check changes its result value
================================================================

.. contents:: Table of Contents
   :depth: 10

Let's describe how the SCA scanner detects and reports changes in the check results using a simple use case.

To configure the execution of the *SCA* module with a policy file, it is necessary to set up a block as follows:

.. code-block:: none

    <sca>
      <enabled>no</enabled>
      <scan_on_start>yes</scan_on_start>
      <interval>12h</interval>
      <skip_nfs>yes</skip_nfs>

      <policies>
          <policy>cis_debian8_L2.yml</policy>
      </policies>
    </sca>

The default path for policies is */var/ossec/ruleset/sca* on Unix environments and *C:\\Program files (x86)\\ossec-agent\\ruleset\\sca* on Windows.

In this case, just the Debian Linux ``cis_debian7_L2.yml`` policy file has been set.
Also, given that ``scan_on_start`` is set to yes, the module will perform a scan right after starting Wazuh.

Results of the first scan are reported and alerts such as the following one are fired for every check:

.. code-block:: none

    ** Alert 1556631403.62082: - sca,gdpr_IV_35.7.d
    2019 Apr 30 06:36:43 ubuntu->sca
    Rule: 19008 (level 3) -> 'CIS benchmark for Debian/Linux: Ensure IPv4 forwarding is disabled'
    {"type":"check","id":1518747324,"policy":"CIS benchmark for Debian/Linux","policy_id":"cis_debian8_L2","check":{"id":5031,"title":"Ensure IPv4 forwarding is disabled","description":"The net.ipv4.ip_forward flag is used to tell the system whether it can forward packets or not.","rationale":"Setting the flag to 0 ensures that a system with multiple interfaces (for example, a hard proxy), will never be able to forward packets, and therefore, never serve as a router.","remediation":"Set the following parameter in /etc/sysctl.conf or a /etc/sysctl.d/* file: net.ipv4.ip_forward = 0","compliance":{"cis_csc":"5.1","cis":"3.1.1"},"rules":["f:/proc/sys/net/ipv4/ip_forward -> 1;"],"file":"/proc/sys/net/ipv4/ip_forward","result":"passed"}}
    sca.type: check
    sca.scan_id: 1518747324
    sca.policy: CIS benchmark for Debian/Linux
    sca.check.id: 5031
    sca.check.title: Ensure IPv4 forwarding is disabled
    sca.check.description: The net.ipv4.ip_forward flag are used to tell the system whether it can forward packets or not.
    sca.check.rationale: Setting the flags to 0 ensures that a system with multiple interfaces (for example, a hard proxy), will never be able to forward packets, and therefore, never serve as a router.
    sca.check.remediation: Set the following parameter in /etc/sysctl.conf or a /etc/sysctl.d/* file: net.ipv4.ip_forward = 0
    sca.check.compliance.cis_csc: 5.1
    sca.check.compliance.cis: 3.1.1
    sca.check.file: ["/proc/sys/net/ipv4/ip_forward"]
    sca.check.result: passed

Those alerts will inform about the initial status of checks. Furthermore, no additional alerts will be
issued unless the state of a check changes between successive scans.

In addition to the check-events, SCA will issue an alert that summarizes the results of policy scans.
In the same way, summary alerts are triggered only upon the first scan, and whenever any check state
changes, as any variation alters the summary information.

.. code-block:: none

    ** Alert 1556631410.84452: - sca,gdpr_IV_35.7.d
    2019 Apr 30 06:36:50 ubuntu->sca
    Rule: 19003 (level 5) -> 'SCA summary: CIS benchmark for Debian/Linux: Score less than 80% (51)'
    {"type":"summary","scan_id":1518747324,"name":"CIS benchmark for Debian/Linux","policy_id":"cis_debian8_L2","file":"cis_debian8_L2.yml","description":"This document provides prescriptive guidance for establishing a secure configuration posture for Debian Linux systems running on x86 and x64 platforms. Many lists are included including filesystem types, services, clients, and network protocols. Not all items in these lists are guaranteed to exist on all distributions and additional similar items may exist which should be considered in addition to those explicitly mentioned.","references":"https://www.cisecurity.org/cis-benchmarks/","passed":16,"failed":15,"invalid":11,"total_checks":42,"score":51.612899780273438,"start_time":1556631391,"end_time":1556631396,"hash":"c84124baa3aa761f279e4360f19584ecd2059493872f0987fedf7d26d7834dad","hash_file":"8db06ce8c56fb7ed50255b5191e3835632b649aeb642c7948c4ac020f1311141","force_alert":"1"}
    sca.type: summary
    sca.scan_id: 1518747324
    sca.policy: CIS benchmark for Debian/Linux
    sca.description: This document provides prescriptive guidance for establishing a secure configuration posture for Debian Linux systems running on x86 and x64 platforms. Many lists are included including filesystem types, services, clients, and network protocols. Not all items in these lists are guaranteed to exist on all distributions and additional similar items may exist which should be considered in addition to those explicitly mentioned.
    sca.policy_id: cis_debian8_L2
    sca.passed: 16
    sca.failed: 15
    sca.invalid: 11
    sca.total_checks: 42
    sca.score: 51
    sca.file: cis_debian8_L2.yml

If we focus in check 5031 (whose alert appears above), we can see it verifies that IP forwarding is
disabled by checking the contents of file */proc/sys/net/ipv4/ip_forward*.

By enabling IPv4 forwarding,

.. code-block:: bash

    echo "1" > /proc/sys/net/ipv4/ip_forward


The next SCA scan for that policy generates the following alert:

.. code-block:: none

    ** Alert 1556641576.310451: - sca,gdpr_IV_35.7.d
    2019 Apr 30 09:26:16 ubuntu->sca
    Rule: 19011 (level 9) -> 'CIS benchmark for Debian/Linux: Ensure IPv4 forwarding is disabled: Status changed from passed to failed'
    {"type":"check","id":25201596,"policy":"CIS benchmark for Debian/Linux","policy_id":"cis_debian","check":{"id":5031,"title":"Ensure IPv4 forwarding is disabled","description":"The net.ipv4.ip_forward flag is used to tell the system whether it can forward packets or not.","rationale":"Setting the flag to 0 ensures that a system with multiple interfaces (for example, a hard proxy), will never be able to forward packets, and therefore, never serve as a router.","remediation":"Set the following parameter in /etc/sysctl.conf or a /etc/sysctl.d/* file: net.ipv4.ip_forward = 0","compliance":{"cis_csc":"5.1","cis":"3.1.1"},"rules":["f:/proc/sys/net/ipv4/ip_forward -> 1;"],"file":"/proc/sys/net/ipv4/ip_forward","result":"failed"}}
    sca.type: check
    sca.scan_id: 25201596
    sca.policy: CIS benchmark for Debian/Linux
    sca.check.id: 5031
    sca.check.title: Ensure IPv4 forwarding is disabled
    sca.check.description: The net.ipv4.ip_forward flag are used to tell the system whether it can forward packets or not.
    sca.check.rationale: Setting the flags to 0 ensures that a system with multiple interfaces (for example, a hard proxy), will never be able to forward packets, and therefore, never serve as a router.
    sca.check.remediation: Set the following parameter in /etc/sysctl.conf or a /etc/sysctl.d/* file: net.ipv4.ip_forward = 0
    sca.check.compliance.cis_csc: 5.1
    sca.check.compliance.cis: 3.1.1
    sca.check.file: ["/proc/sys/net/ipv4/ip_forward"]
    sca.check.result: failed
    sca.check.previous_result: passed

The level 9 alert shows how the check has changed from **passed** to **failed**. This state is updated on the
manager side and the last result scanned is available from the SCA tab in the Wazuh app, which finishes the path
that began with the host being scaned.

.. thumbnail:: ../../../images/sca/SCA-ip-forward-check.png
    :title: Alert about IP forwarding check
    :align: center
    :width: 100%
