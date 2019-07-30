---
title: Android接听挂断电话
date: 2019-07-30 17:16:20
tags:
---

### 完整代码

``` bash
package com.linkplay.phonedemo;

import android.Manifest;
import android.annotation.SuppressLint;
import android.app.Activity;
import android.content.Context;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.IBinder;
import android.support.v4.app.ActivityCompat;
import android.telecom.TelecomManager;
import android.telephony.TelephonyManager;
import android.util.Log;

import com.android.internal.telephony.ITelephony;

import java.lang.reflect.Method;


public class PhoneUtil {

    private static final String TAG = Class.class.getSimpleName();

    private static void permissionCheck(Activity activity, String[] PERMISSIONS) {
        if (!hasPermissions(activity, PERMISSIONS)) {
            ActivityCompat.requestPermissions(activity, PERMISSIONS, 0);
        }
    }

    private static boolean hasPermissions(Context context, String... permissions) {
        if (android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.M && context != null && permissions != null) {
            for (String permission : permissions) {
                if (ActivityCompat.checkSelfPermission(context, permission) != PackageManager.PERMISSION_GRANTED) {
                    return false;
                }
            }
        }
        return true;
    }


    public static void endCall(Activity context) {

        Log.i(TAG, "Build.VERSION.SDK_INT == " + Build.VERSION.SDK_INT);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {

            permissionCheck(context, new String[]{Manifest.permission.ANSWER_PHONE_CALLS});

            TelecomManager tm = (TelecomManager)
                    context.getSystemService(Context.TELECOM_SERVICE);
            if (tm == null) {
                Log.e(TAG, "TelecomManager == null");
                return;
            }
            if (ActivityCompat. checkSelfPermission(context, Manifest.permission.ANSWER_PHONE_CALLS)
                    != PackageManager.PERMISSION_GRANTED) {
                Log.e(TAG, "没有权限： " + Manifest.permission.ANSWER_PHONE_CALLS);
                return;
            }
            tm.endCall();
            return;
        }

        permissionCheck(context, new String[]{Manifest.permission.CALL_PHONE});
        try {
            @SuppressLint("PrivateApi")
            Method method = Class.forName("android.os.ServiceManager").getMethod("getService", String.class);
            method.setAccessible(true);
            IBinder binder = (IBinder) method.invoke(null, new Object[]{Context.TELEPHONY_SERVICE});
            ITelephony telephony = ITelephony.Stub.asInterface(binder);
            telephony.endCall();
        } catch (NoSuchMethodException e) {
            Log.d(TAG, "", e);
        } catch (ClassNotFoundException e) {
            Log.d(TAG, "", e);
        } catch (Exception e) {
            Log.d(TAG, "", e);
        }
    }


    public static void acceptCall(Activity activity) {
        Log.i(TAG, "Build.VERSION.SDK_INT == " + Build.VERSION.SDK_INT);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {

            permissionCheck(activity, new String[]{Manifest.permission.ANSWER_PHONE_CALLS});

            TelecomManager tm = (TelecomManager) activity.getSystemService(Context.TELECOM_SERVICE);
            if (tm == null) {
                Log.e(TAG, "TelecomManager == null");
                return;
            }
            if (ActivityCompat.checkSelfPermission(activity, Manifest.permission.ANSWER_PHONE_CALLS) != PackageManager.PERMISSION_GRANTED) {
                Log.e(TAG, "没有权限： " + Manifest.permission.ANSWER_PHONE_CALLS);
                return;
            }
            tm.acceptRingingCall();
            return;
        }

        permissionCheck(activity, new String[]{Manifest.permission.MODIFY_PHONE_STATE});
        try {
            @SuppressLint("PrivateApi")
            Method method = Class.forName("android.os.ServiceManager").getMethod("getService", String.class);
            method.setAccessible(true);
            IBinder binder = (IBinder) method.invoke(null, new Object[]{Context.TELEPHONY_SERVICE});
            ITelephony telephony = ITelephony.Stub.asInterface(binder);
            telephony.answerRingingCall();
        } catch (NoSuchMethodException e) {
            Log.d(TAG, "", e);
        } catch (ClassNotFoundException e) {
            Log.d(TAG, "", e);
        } catch (Exception e) {
            Log.d(TAG, "", e);
        }
    }
}

```