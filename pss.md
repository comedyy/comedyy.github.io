### pss 内存


```
import android.app.ActivityManager;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.util.Log;
import com.unity3d.player.UnityPlayer;

import static android.content.Context.ACTIVITY_SERVICE;
public static long getProcessSize() {
    ActivityManager activityManager = (ActivityManager) UnityPlayer.currentActivity.getSystemService(ACTIVITY_SERVICE);
    android.os.Debug.MemoryInfo[] memInfos = activityManager.getProcessMemoryInfo(new int[] { android.os.Process.myPid() });
    return memInfos[0].getTotalPss() * 1024;
}

```



```
#import <mach/mach.h>
extern "C" {
    float GetPhysMemory()
	{
        task_vm_info_data_t vmInfo;
        mach_msg_type_number_t count2 = TASK_VM_INFO_COUNT;
        kern_return_t kernReturn = task_info(mach_task_self(), TASK_VM_INFO, (task_info_t) &vmInfo, &count2);
        if (kernReturn == KERN_SUCCESS)
        {
            int64_t memoryUsageInByte = (int64_t) vmInfo.phys_footprint;
            return memoryUsageInByte/1024.0/1024.0;
        }
        else{
            return 0;
        }
    }
}

```