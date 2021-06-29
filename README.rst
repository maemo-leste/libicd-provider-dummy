=====================
libicd-provider-dummy
=====================

Example dummy service module.

As of writing, the srv_provider API of ICD2 is very underdocumented, so this is
meant to assist in exploring and cleaning up the API and ICD2 code, as well as
discover how this is used from the users perspective.

If libicd-network-dummy is installed, this should work, and you should see
"Dummy Provider" in the connection dialog. It is not clear how me exactly how a
specific network is picked, yet. If we do the same for wifi, we want to be able
to pick a service provider *AND* a specific IAP. We need to figure out how this
works in the UI.

To investigate
==============


FAQ
===

Can providers be chained?
-------------------------

From icd/icd_srv_provider.c's icd_srv_provider_connect, it looks like they
likely cannot be chained. Thus there can be only one provider. (The function
searches for a specific module and calls the connect() function there)

But ... what does icd_srv_provider_has_next do?

I saw this when connecting to my wlan provider:

Jun 29 13:31:54 localhost icd2 0.98[20661]: calling service provider module
Jun 29 13:31:54 localhost icd2 0.98[20661]: dummy_connect: 7de4b188-3ef1-4859-80d8-676b0682db85
Jun 29 13:31:54 localhost icd2 0.98[20661]: service module connect callback
Jun 29 13:31:54 localhost icd2 0.98[20661]: No service provider module to call

So it looks like you can actually chain services


Preferred service
=================

connui-common: connui/iap-common: iap_common_get_preferred_service lists these
keys:

* /system/osso/connectivity/srv_provider/preferred_type
* /system/osso/connectivity/srv_provider/preferred_id


Status icons
============

It might be possible that setting the following keys will show override some
icons in the status bar, but I have not gotten it to work yet:

$ gconftool -t string -s /system/osso/connectivity/srv_provider/DUMMY/custom_ui/dummy-provider/type_statusbar_dimmed_icon_name general_wlan
$ gconftool -t string -s /system/osso/connectivity/srv_provider/DUMMY/custom_ui/dummy-provider/type_statusbar_icon_name general_wlan

This might also do something for getting the icon of an IAP:

$ gconftool -t string -s /system/osso/connectivity/srv_provider/DUMMY/custom_ui/dummy-provider/type_icon_name general_wlan


Setup in gconf
==============

$ gconftool -R /system/osso/connectivity/srv_provider
 /system/osso/connectivity/srv_provider/DUMMY:
  module = libicd_provider_dummy.so
  network_type = [DUMMY,WLAN_INFRA]

$ gconftool -s /system/osso/connectivity/srv_provider/DUMMY/network_type --type list --list-type string '[DUMMY,WLAN_INFRA]'



SORTME
=======

* iap_common_set_service_properties_for_iap in connui-common connui/iap-common.c
  - seems to be code to set certain gtk container/label properties
* iap_common_set_service_properties

TODO
====

Searching for this in connui-* might surface more properties:

iap_common_get_service_properties

Like:

        if (scan_entry->network.service_id && *scan_entry->network.service_id)
        {
          iap_common_get_service_properties(scan_entry->network.service_type,
                                            scan_entry->network.service_id,
                                            "scan_results", &scan_results,
                                            NULL);


    iap_common_get_service_properties(service_type, service_id,
                                      "icon_name", &icon_name,
                                      "markup", &format,
                                      NULL);
    if (format)
      label_text = g_strdup_printf(format, service_text);
    else
      label_text = g_strdup(service_text);


      iap_common_get_service_properties(service_type, service_id,
                                        "gettext_catalog", &domainname,
                                        "name", &msgid,
                                        NULL);

    iap_common_get_service_properties(entry->service_type,
                                      entry->service_id,
                                      "gettext_catalog", &domainname,
                                      "name", &msgid,
                                      NULL);

