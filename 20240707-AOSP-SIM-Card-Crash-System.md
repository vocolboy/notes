# AOSP SIM card crash system

# Same Issue
* https://issuetracker.google.com/issues/322484741

# Reappear Issue
Using AOSP-13.0.0_r78 synced directly from Google for compilation,
I encountered this issue when input a SIM card.

```
2024-07-07 18:58:48.799   326-326   init                    pid-326                              E  Unable to set property 'persist.vendor.radio.call_waiting_for_sync_0' from uid:10101 gid:10101 pid:8460: SELinux permission check failed
2024-07-07 18:58:48.815  3124-7334  DynamicModuleDownloader com.google.android.gms               E  Zapp module request failed: 1,4
2024-07-07 18:58:49.033  8551-8570  SHANNON_IMS             com.shannon.imsservice               E  0010 [CONF] External configuration xml file not exist in vendor/etc. Name: common_configuration.xml (BaseConfigurationReader%getInputStreamVendor:293)
2024-07-07 18:58:49.055  8551-8570  SHANNON_IMS             com.shannon.imsservice               E  0022 [CONF] External configuration xml file not exist in vendor/etc. Name: sim_configurations/sim_configuration_452.xml (BaseConfigurationReader%getInputStreamVendor:293)
2024-07-07 18:58:49.069  8551-8638  SHANNON_IMS             com.shannon.imsservice               E  0032 [CONF] External configuration xml file not exist in vendor/etc. Name: emergency_type.xml (BaseConfigurationReader%getInputStreamVendor:293)
2024-07-07 18:58:49.077  8551-8638  SHANNON_IMS             com.shannon.imsservice               E  0049 [CONF] External configuration xml file not exist in vendor/etc. Name: operator_config.xml (BaseConfigurationReader%getInputStreamVendor:293)
2024-07-07 18:58:49.082  8551-8638  SHANNON_IMS             com.shannon.imsservice               E  0055 [CONF] External configuration xml file not exist in vendor/etc. Name: sim_operator_list.xml (BaseConfigurationReader%getInputStreamVendor:293)
2024-07-07 18:58:49.507  1382-1445  system_server           system_process                       E  No package ID 7f found for ID 0x7f08034a.
2024-07-07 18:58:49.507  1382-1445  system_server           system_process                       E  No package ID 7f found for ID 0x7f14076f.
2024-07-07 18:58:49.679  8551-8685  AndroidRuntime          com.shannon.imsservice               E  FATAL EXCEPTION: pool-13-thread-1
                                                                                                    Process: com.shannon.imsservice, PID: 8551
                                                                                                    java.lang.RuntimeException: failed to set system property (check logcat for reason)
                                                                                                    	at android.os.SystemProperties.native_set(Native Method)
                                                                                                    	at android.os.SystemProperties.set(SystemProperties.java:235)
                                                                                                    	at com.shannon.imsservice.ut.ImsUt.setCallWaitingSyncProperty(ImsUt.java:1353)
                                                                                                    	at com.shannon.imsservice.ut.ImsUt.refreshCallWaiting(ImsUt.java:1202)
                                                                                                    	at com.shannon.imsservice.ut.ImsUt.lambda$refreshCallWaitingAsync$27(ImsUt.java:1119)
                                                                                                    	at com.shannon.imsservice.ut.ImsUt.$r8$lambda$V3-4-1Ak7OCWx7C1l3QhRu_06mA(Unknown Source:0)
                                                                                                    	at com.shannon.imsservice.ut.ImsUt$$ExternalSyntheticLambda15.run(Unknown Source:4)
                                                                                                    	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1137)
                                                                                                    	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:637)
                                                                                                    	at java.lang.Thread.run(Thread.java:1012)
```

# Fix Step
Inspired by https://github.com/minaripenguin/android_hardware_google_gs101-sepolicy

```diff
device/google/gs101-sepolicy/system_ext/private/platform_app.te

# allow systemui to set boot animation colors
set_prop(platform_app, bootanim_system_prop);
+ set_prop(platform_app, shannon_ims_service_prop);

# allow systemui to access fingerprint
hal_client_domain(platform_app, hal_fingerprint)
```

```diff
device/google/gs101-sepolicy/system_ext/private/property_contexts

# Fingerprint (UDFPS) GHBM/LHBM toggle
persist.fingerprint.ghbm    u:object_r:fingerprint_ghbm_prop:s0    exact    bool

# Boot animation dynamic colors
persist.bootanim.color1     u:object_r:bootanim_system_prop:s0     exact    int
persist.bootanim.color2     u:object_r:bootanim_system_prop:s0     exact    int
persist.bootanim.color3     u:object_r:bootanim_system_prop:s0     exact    int
persist.bootanim.color4     u:object_r:bootanim_system_prop:s0     exact    int

+ # Properties for euicc
+ persist.modem.esim_profiles_exist            u:object_r:esim_modem_prop:s0    exact    string

+ # Telephony
+ telephony.ril.silent_reset    u:object_r:telephony_ril_prop:s0    exact    bool

+ # ShannonIms
+ persist.vendor.radio.call_waiting_for_sync_0    u:object_r:shannon_ims_service_prop:s0    exact    int
+ persist.vendor.radio.call_waiting_for_sync_1    u:object_r:shannon_ims_service_prop:s0    exact    int
```

```diff
device/google/gs101-sepolicy/system_ext/public/property.te

# Fingerprint (UDFPS) GHBM/LHBM toggle
system_vendor_config_prop(fingerprint_ghbm_prop)

+ # eSIM properties
+ system_vendor_config_prop(esim_modem_prop)
 
+ # Telephony
+ system_public_prop(telephony_ril_prop)

+ userdebug_or_eng(`
+   set_prop(shell, telephony_ril_prop)
+ ')

+ # ShannonIms properties
+ system_public_prop(shannon_ims_service_prop)
```