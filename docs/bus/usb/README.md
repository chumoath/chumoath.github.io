#  USB

### 1、linux imx6ull USB

```c
imx6ull usb:
drivers/usb/chipidea/ci_hdrc_imx.c
	compatible: "fsl,imx6ul-usb"  "fsl,imx27-usb"
	
	ci_hdrc_imx_probe -> ci_hdrc_add_device
		// 添加新的platform设备: "ci_hdrc"
		platform_device_alloc("ci_hdrc", id);
		platform_device_add(pdev);

drivers/usb/chipidea/core.c
drivers/usb/chipidea/udc.c
	ci_hdrc_probe -> OTG/HOST: ci_hdrc_host_init
				  -> OTG/PERIPHERAL: ci_hdrc_gadget_init -> udc_start
									 ci_hdrc_otg_init    => INIT_WORK(&ci->work, ci_otg_work);
														 => ci->wq = create_freezable_workqueue("ci_otg");
				  -> ci_role_start -> host_start/udc_id_switch_for_device
			
			host_start -> __usb_create_hcd
			           -> usb_add_hcd
			
			udc_start -> init_eps -> list_add_tail(&hwep->ep.ep_list, &ci->gadget.ep_list);
					  -> usb_add_gadget_udc -> usb_add_gadget 
												-> list_add_tail(&udc->list, &udc_list);
			
			中断：
				ci_hdrc_probe -> ci_irq_handler
					OTG: ci_otg_queue_work -> queue_work(ci->wq, &ci->work) => ci_otg_work
					
					ci_role(ci)->irq(ci); => udc_irq
													=> ci->driver->resume()
													=> ci->driver->suspend()
													=> isr_tr_complete_handler => isr_setup_packet_handler => ci->driver->setup()
					
gadget:
drivers/usb/gadget/udc/core.c
	gadget_dev_desc_UDC_store -> usb_gadget_probe_driver 
	usb_composite_probe -> usb_gadget_probe_driver
			-> list_for_each_entry(udc, &udc_list, list)
			-> udc_bind_to_driver
				-> usb_gadget_driver.bind()
				-> usb_gadget_udc_start => udc->gadget->ops->udc_start()
				-> usb_udc_connect_control: udc->vbus
										   -> usb_gadget_connect    => gadget->ops->pullup(gadget, 1);
																	
										   -> usb_gadget_disconnect => gadget->ops->pullup(gadget, 0);
																	=> gadget->udc->driver->disconnect(gadget);

初始化: ci_hdrc_probe -> ci_hdrc_gadget_init -> udc_start -> init_eps -> hwep.ep.ops = &usb_ep_ops;
使用：drivers/usb/gadget/udc/core.c: usb_ep_queue => ep->ops->queue() -> ep_queue
                                    usb_ep_dequeue => ep->ops->dequeue() -> ep_dequeue
    
    composite_bind -> composite_dev_prepare -> usb_ep_alloc_request => ep->ops->alloc_request() -> ep_alloc_request
    composite_bind -> composite_dev_prepare -> usb_ep_free_request => ep->ops->free_request() -> ep_free_request
    
    usb_gadget_probe_driver -> udc_bind_to_driver -> usb_gadget_udc_start => udc->gadget->ops->udc_start -> ci_udc_start -> usb_ep_enable => ep->ops->enable() -> 
    
function: drivers/usb/gadget/function/f_fs.c
    ffs_fs_type -> ffs_fs_init_fs_context -> ffs_fs_context_ops -> ffs_fs_context_ops -> ffs_fs_get_tree -> ffs_sb_fill 
    	-> ffs_sb_create_file -> ffs_ep0_operations -> ffs_ep0_write -> ffs_epfiles_create 
    	-> ffs_sb_create_file -> ffs_epfile_operations -> ffs_epfile_ioctl 
    		-> usb_ep_fifo_status => ep->ops->fifo_status()
            -> usb_ep_fifo_flush => ep->ops->fifo_flush() -> ep_fifo_flush

fs-function: drivers/usb/gadget/legacy/inode.c
    init -> gadgetfs_type -> gadgetfs_init_fs_context -> gadgetfs_context_ops -> gadgetfs_get_tree -> gadgetfs_fill_super 
    	-> gadgetfs_create_file -> ep0_operations -> dev_config(.write) 
    		-> usb_gadget_probe_driver -> gadgetfs_driver -> gadgetfs_bind -> activate_ep_files 
    			-> gadgetfs_create_file -> ep_io_operations -> ep_ioctl 
    				-> usb_ep_fifo_status => ep->ops->fifo_status()
    				-> usb_ep_fifo_flush => ep->ops->fifo_flush() -> ep_fifo_flush

传输: usb_ep_alloc_request -> req.buf (数据) req.length (长度) req.complete (完成后回调) req.ctx.ep (端点)
	 usb_ep_queue -> ep_queue -> _ep_queue -> req.actual = 0
    									-> _hardware_enqueue 
    										-> usb_gadget_map_request_by_dev (dma_map_single: 将req.buf虚拟地址map为DMA地址 -> req.data)
                                            -> prepare_td_for_non_sg (每次TD_PAGE_COUNT 5个page，req.actual为步进值)
    										  -> while (rest > 0) {
                                                count = min(hwreq->req.length - hwreq->req.actual, (unsigned int)(pages * CI_HDRC_PAGE_SIZE));
    											-> add_td_to_list
    												-> node = kzalloc(sizeof(struct td_node), GFP_ATOMIC);
													-> 分配硬件ci_hw_td结构需要的DMA地址: node->ptr = dma_pool_zalloc(hwep->td_pool, GFP_ATOMIC, &node->dma);
													-> 获取当前数据的DMA地址: temp = (u32) (hwreq->req.dma + hwreq->req.actual);
													-> 将数据的物理页放到ci_hw_td.ptr.page: node->ptr->page[0] = cpu_to_le32(temp);
														-> for (i = 1; i < TD_PAGE_COUNT; i++) { u32 page = temp + i * CI_HDRC_PAGE_SIZE(4096); node->ptr->page[i] = cpu_to_le32(page);}
													-> 数据步进: hwreq->req.actual += length;
													-> INIT_LIST_HEAD(&node->td);
													-> 将当前td_node放到hwreq->tds链表: list_add_tail(&node->td, &hwreq->tds);
                                                 -> rest -= count;
                                        -> 将当前req放到EP队列: list_add_tail(&hwreq->queue, &hwep->qh.queue);

发送: ci_hdrc_gadget_init -> rdrv->irq = udc_irq;
    	-> ci_irq_handler => ci_role(ci)->irq(ci) => udc_irq => isr_tr_complete_handler => isr_tr_complete_low
            -> list_for_each_entry_safe(hwreq, hwreqtemp, &hwep->qh.queue, queue)
            	-> _hardware_dequeue(hwep, hwreq);  发送数据
					-> list_for_each_entry_safe(node, tmpnode, &hwreq->tds, td) 遍历req的tds
				-> list_del_init(&hwreq->queue);
				-> usb_gadget_giveback_request(&hweptemp->ep, &hwreq->req);
					=> req->complete(ep, req);
```

### 2、gadget interface

```c
struct usb_gadget_driver
struct semaphore
struct usb_ep
struct usb_request
wait_queue_head_t
spin_lock_init
init_MUTEX
init_waitqueue_head
wake_up_interruptible
wait_event_interruptible_timeout
kzalloc
down
up
atomic64_inc
timer_after
msecs_to_jiffies
kobject_uevent
kobject_name
ALIGN
__ffs
__fls
usb_gadget_probe_driver
usb_gadget_unregister_driver
usb_gadget_connect
usb_gadget_disconnect
schedule_work
usb_endpoint_maxp
usb_ep_queue
usb_ep_dequeue
```

