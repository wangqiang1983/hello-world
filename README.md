# hello-world
just another repository 


this is my edit file

## if you have any questions, please contact me with my [github](https://github.com/wangqiang1983)

- [ ] _first job_
- [ ] second job
- [x] **already done**


**already done**


Name | Age | Address
-------|--------|----------
Wangqiang| 35 | Hefei
Wangjiaming | 35 | Anhui


# This is new msg for modified


----
****
## this is a new branch



```mermaid
sequenceDiagram
    participant RX as thread_rx_gb_read(group)
    participant GR as request_group_read.group_read_grp_queue[grp_mpu_id]
    participant D as thread_dispatch_cmd_gb_read
    participant L as thread_gb_read_launch(ch)
    participant F as thread_gb_read_fetch_trans(ch)
    participant M as thread_gb_read_fill_mlane_fifo(ch)

    Note over RX,GR: 结构体层：gb_read_req_fields -> turn/group_read -> gb_read_req_fields_v

    RX->>RX: wait_for_gb_read_req()
    RX->>RX: req_arr[group][mpu].push_back(gb_read_req_fields)

    alt req.gb_read_last == 1
        RX->>GR: 找/建 turn (按 grp_mpu_id)
        RX->>GR: turn.turn_map[global_mpu_id] = req_arr[group][mpu]
        RX->>GR: arrived_mpu_id.insert(global_mpu_id)
        RX->>GR: expected_mpu_id (首次建turn时由 get_all_ids 得到)

        RX->>RX: checkAllAddrSame(turn_map)

        alt turn.is_ready()
            RX->>RX: 组装 gb_read_req_fields_v(tt)
            RX->>D: request_queue_gb_read.write(tt)
            RX->>GR: qhandle.erase(ready turn)
        else 未就绪
            RX-->>RX: 等其他MPU同轮补齐
        end
    else 非last
        RX-->>RX: 继续积累该MPU本轮片段
    end

    D->>D: 从 request_queue_gb_read 取 reqs
    alt channel_map已有 grp_mpu_id
        D->>L: ch_gb_read[ch_id].cmd_fifo.write(reqs)
    else 分配空闲读通道
        D->>L: 绑定 channel_map + write(cmd_fifo) + notify(evt_gb_read_cmd_go)
    end

    L->>L: 计算 rd_num(由 addr_strobe)
    L->>L: 给每个 req 标 turn_id/rd_num
    L->>L: 拆地址并发 read_gb_re()
    L->>F: rd_fifo.write(rd_fifo_ele_t{req,...})

    F->>F: fetch_data_rd()
    F->>F: req.rd_cnt递增（按turn内返回顺序）
    F->>M: wr_fifo.write(wr_data_ele_t{data, req, idx})

    M->>M: 组 packet_info_t，ml_gbread_last=(rd_cnt==rd_num)&&gb_read_last
    M->>M: 按 gb_read_grp_cpy 广播到对应 mlane_cmd_out_fifo
    alt cmd_fifo/rd_fifo/wr_fifo 都空且 gb_read_ostd==0
        M->>M: channel reset + occupied=false + evt_gb_read_cmd_done
    end

```
