Issues
======

``S001`` 2D 영상 저장하는 interval이 지정값보다 현저히 늘어남 (20240910)
--------------------------------------------------------------------------------------------------
2D 영상 촬영 interval을 1초로 했는데,

+ DAQ 시작 초기에는 5-7초 interval로 촬영이 되고
+ 그 interval이 점점 길어져서 후에는 30초 까지 늘어남

Script를 실행했다가 ``Ctrl+C`` 로 강제 종료하고, 또 script를 실행하고 ``Ctrl+C`` 로 강제 종료하고 하는 것을 반복했음.

Interval이 늘어나는 것이 누적된 2D 영상 촬영 횟수와 상관이 있는 것으로 보임

    .. image:: images/0010_intervals.png
        :align: center


    .. code-block:: Python
        :emphasize-lines: 22

        cond = {
            'exptime': 100,          # Expose time in milliseconds
            'numprj': 1181,         # Number of frames
            'pathname': 'cell5_test',     # Pathname

            #'numiter': 50,          # Number of scan iteration
            
            'x2origin': -2990.00,   # Sample stage origin
            'pullback': 8000,     # X2_Pullback distance
            'numff': 10,             # Number of flat fields
            'numdf': 10,             # Number of dark fields
            'default_speed': 4,     # Sample stage speed
            'uuid': str(uuid.uuid4()), # Scan uuid
            'stop_angle': 200,      # Target angle
        }

        cond_2d = {
            'exptime': cond['exptime'],          # Expose time in milliseconds
            'numprj': 45,         # Number of frames
            #'numprj': 15,         # Number of frames test
            'pathname': 'cell5_test_2d',     # Path name
            'interval': 1000,  #ms
        ##########################################################    
            'numff' : 0,
            'numdf' : 0,
            'uuid': str(uuid.uuid4()), # Scan uuid
            'default_speed': 4,     # Sample stage speed
            'user_filename': 'temp'
        }

        import datetime
        import pandas as pd

        cond['scan_speed'] = 0.5 * 60000 / (cond['numprj']) / (cond['exptime']+30) # Rot./min.
        cond['start_angle'] =  -5 # Start anglecond['start_angle'] =  (-1)*180/(cond['numprj']-1)*5 + 0 # Start angle

        logger.info(f"Scan parameters : {cond}")

        # Load time list
        data = pd.read_excel('C:\\codes\\bl6c_daq\\sample.xlsx')
        times = data['time']

        time_list = []
        for it in times:
            time_list.append(it.strftime("%H:%M"))
            
        print(f"time_list : {time_list}")

        def finalize_pco(): #script stop code
            # Close shutter
            yield from bps.mv(shutter, 0)
            # Set exposure time in seconds
            yield from bps.mv(pco.cam.acquire_time, cond['exptime']/1000,
                            pco.cam.trigger_mode, 0) # Internal trigger
            yield from bps.abs_set(sample.rot_stop, 1)
            yield from bps.mv(sample.rot.velocity, cond['default_speed'])
            yield from bps.mv(sample.rot, 0)
            yield from bps.mv(sample.x2, cond['x2origin'])
            
        
        @finalize_decorator(finalize_pco)
        def time_trigger():
            """
            """

            # Enable saving
            pco.save_image(True)
            # Stats calculation is not needed
            pco.enable_stats(False)


            logger.info(f"Moving sample.rot to {cond['start_angle']}, x2 to {cond['x2origin']}")
            yield from bps.mv(sample.rot.velocity, cond['default_speed'])
            yield from bps.mv(sample.rot, cond['start_angle'],
                            sample.x2, cond['x2origin'])
                            
            start_position = yield from bps.rd(sample.rot)
            
            logger.info(f"Start CT scan at {datetime.datetime.now()}, motor start position : {start_position}")
            
            ct = 1
            # CT scan
            # for _ in range(cond['numiter']):
            for _ in range(len(time_list)):
                
                ## Wait for specfied time 
                while True:
                    current_time = datetime.datetime.now().strftime("%H:%M")
                    print(f"current_time : {current_time}")
                    if current_time in time_list:
                        time_list.remove(current_time)
                        logger.info(f"Scan triggered : {current_time}")
                        break
                    else:
                        yield from bps.sleep(1)
                
                yield from bps.mv(shutter, 1)
                yield from bps.mv(sample.rot, cond['start_angle'])
                yield from bps.abs_set(sample.rot_stop, 1)
                yield from bps.sleep(1) # 1 second

                start_position = yield from bps.rd(sample.rot)
                logger.info(f"Start CT scan at {datetime.datetime.now()}, motor start position : {start_position}, velocity : {cond['scan_speed']}")
                
                yield from bps.mv(sample.rot.velocity, cond['scan_speed'],
                                pco.cam.num_images, cond['numprj'],
                                pco.cam.trigger_mode, 4) # External trigger mode
                yield from bps.abs_set(sample.rot, 200)
                yield from bps.sleep(cond['exptime']/1000*5)
                yield from bp.count([pco], md={'reason' : 'CT scan',
                                            'uuid' : cond['uuid'],
                                            'settings': cond})
                stop_position = yield from bps.rd(sample.rot)
                yield from bps.mv(shutter, 0) 

                logger.info(f"Finished CT scan at {datetime.datetime.now()}, stop position : {stop_position}")
                yield from bps.abs_set(sample.rot_stop, 1)
                yield from bps.sleep(1)
                yield from bps.mv(sample.rot.velocity, cond['default_speed'])
                yield from bps.sleep(1)
                yield from bps.mv(sample.rot, 0)
                yield from bps.sleep(1)
                
                logger.info(f"Finished CT scan at {datetime.datetime.now()}, stop position : {stop_position}")
                logger.info(f"{ct} Finished!")
                ct = ct+1
                        
                yield from bps.mv(sample.rot, 0)
                yield from bps.sleep(1)
                yield from bps.mv(sample.rot, 0)
                
                # Set exposure time in seconds
                yield from bps.mv(pco.cam.acquire_time, cond['exptime']/1000,
                                pco.cam.num_images, 1,
                                pco.cam.trigger_mode, 0) # Internal trigger
                                
                #yield from bps.abs_set(sample.rot, 90)
            
                # Dark fields
                logger.info(f"Measure Dark field : {cond['numdf']} frames")
                yield from bps.mv(shutter, 0) # Close shutter
                yield from bp.count([pco],
                                    num=cond['numdf'],
                                    md={'reason': 'dark-field',
                                        'uuid': cond['uuid'],
                                        'settings': cond})

                # Flat fields
                logger.info(f"Measure Flat field : {cond['numff']} frames")
                
                yield from bps.mvr(sample.x2, cond['pullback'])
                #yield from bps.mvr(sample.wireless_x, cond['pullback'])
                #yield from bps.mvr(sample.z, cond['pullback'])
                
                yield from bps.mv(shutter, 1)    
                yield from bp.count([pco],
                                    num=cond['numff'],
                                    md={'reason': 'flat-field',
                                        'uuid': cond['uuid'],
                                        'settings': cond})
                yield from bps.mv(shutter, 0) 

                yield from bps.mv(sample.rot, cond['start_angle'])
                yield from bps.sleep(1) # 1 second    
                
                yield from bps.mvr(sample.x2, -1*cond['pullback'])
                #yield from bps.mvr(sample.wireless_x, -1*cond['pullback'])
                #yield from bps.mvr(sample.z, -1*cond['pullback'])
                
                # 2Dim
                ##############################################
                yield from bps.mv(sample.rot,0)
                yield from bps.sleep(1)
                yield from bps.mv(sample.rot,0)
                yield from bps.sleep(1)
                yield from bps.mv(sample.rot,0)
                yield from bps.sleep(1)

                for _ in range(cond_2d['numprj']):
                    start_time = ttime.time()
                    yield from bps.mv(shutter, 1,
                                    pco.cam.num_images, 1)
                    yield from bps.sleep(1)                  
                    yield from bp.count([pco],
                                        num=1,
                                        md={'reason': 'CT scan',
                                            'uuid': cond_2d['uuid'],
                                            'settings': cond_2d})
                    delta = ttime.time() - start_time
                    yield from bps.mv(shutter, 0)         
                    yield from bps.sleep(cond_2d['interval']/1000)
                ##############################################
                
                cond['uuid'] = str(uuid.uuid4())
                cond_2d['uuid'] = cond['uuid'] #2D 폴더 1개만 생성되는 UUID 초기화
            
        
            pco.save_image(False)
            

        # Run the plan
        #logger.info(f"Start CT scan at {datetime.datetime.now()}")
        RE(time_trigger())

        
실행 log:

    .. code-block:: Python
        :emphasize-lines: 86, 100, 114

        current_time : 02:58
        current_time : 02:58
        current_time : 02:58
        current_time : 02:58
        current_time : 02:58
        current_time : 02:58
        current_time : 02:58
        current_time : 02:58
        current_time : 02:58
        current_time : 02:59
        2024-09-09 02:59:00,279 [daq:INFO] Scan triggered : 02:59

        2024-09-09 02:59:03,931 [daq:INFO] Start CT scan at 2024-09-09 02:59:03.931286, motor start position : -5.0, velocity : 0.1954015501856315

        Transient Scan ID: 5199     Time: 2024-09-09 02:59:16
        Persistent Unique Scan ID: '0c21c892-e5d5-4855-b74f-31ec0857f7fd'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 03:01:51.8 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['0c21c892'] (scan num: 5199)



        2024-09-09 03:02:10,572 [daq:INFO] Finished CT scan at 2024-09-09 03:02:10.572242, stop position : 200.0

        2024-09-09 03:02:16,489 [daq:INFO] Finished CT scan at 2024-09-09 03:02:16.489096, stop position : 200.0

        2024-09-09 03:02:28,919 [daq:INFO] Measure Dark field : 10 frames

        Transient Scan ID: 5200     Time: 2024-09-09 03:02:44
        Persistent Unique Scan ID: '815a194d-8e61-4bd1-ac1a-3cc7d56df179'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 03:02:46.8 |         1005.4 |
        |         2 | 03:02:48.6 |         1005.4 |
        |         3 | 03:02:50.5 |         1005.4 |
        |         4 | 03:02:52.3 |         1005.4 |
        |         5 | 03:02:54.1 |         1005.4 |
        |         6 | 03:02:55.9 |         1005.4 |
        |         7 | 03:02:57.7 |         1005.4 |
        |         8 | 03:02:59.5 |         1005.4 |
        |         9 | 03:03:01.3 |         1005.4 |
        |        10 | 03:03:03.1 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['815a194d'] (scan num: 5200)



        2024-09-09 03:03:13,644 [daq:INFO] Measure Flat field : 10 frames


        Transient Scan ID: 5201     Time: 2024-09-09 03:03:44
        Persistent Unique Scan ID: 'f134794b-57ab-42bb-b694-fd91c2b29fec'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 03:03:46.1 |         1005.4 |
        |         2 | 03:03:47.9 |         1005.4 |
        |         3 | 03:03:49.7 |         1005.4 |
        |         4 | 03:03:51.5 |         1005.4 |
        |         5 | 03:03:53.3 |         1005.4 |
        |         6 | 03:03:55.1 |         1005.4 |
        |         7 | 03:03:56.9 |         1005.4 |
        |         8 | 03:03:58.7 |         1005.4 |
        |         9 | 03:04:00.5 |         1005.4 |
        |        10 | 03:04:02.3 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['f134794b'] (scan num: 5201)





        Transient Scan ID: 5202     Time: 2024-09-09 03:05:03
        Persistent Unique Scan ID: 'c4b3c29e-997f-49d6-bd6f-c83a12192f1e'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 03:05:05.4 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['c4b3c29e'] (scan num: 5202)





        Transient Scan ID: 5203     Time: 2024-09-09 03:05:33
        Persistent Unique Scan ID: 'e2357baf-04e7-4909-8288-7c08ce86b0a0'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 03:05:35.1 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['e2357baf'] (scan num: 5203)





        Transient Scan ID: 5204     Time: 2024-09-09 03:06:03
        Persistent Unique Scan ID: 'dc78081b-0bf4-47b7-b7a4-53651af168cd'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 03:06:05.3 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['dc78081b'] (scan num: 5204)


    .. code-block:: Python
        :emphasize-lines: 83, 97, 111

        current_time : 07:28
        current_time : 07:28
        current_time : 07:28
        current_time : 07:28
        current_time : 07:28
        current_time : 07:28
        current_time : 07:29
        2024-09-09 07:29:00,868 [daq:INFO] Scan triggered : 07:29

        2024-09-09 07:29:05,305 [daq:INFO] Start CT scan at 2024-09-09 07:29:05.305831, motor start position : -5.0, velocity : 0.1954015501856315

        Transient Scan ID: 5631     Time: 2024-09-09 07:29:22
        Persistent Unique Scan ID: '2ce324ed-e549-4e71-93ea-d737ec4b6303'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 07:31:58.0 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['2ce324ed'] (scan num: 5631)



        2024-09-09 07:32:12,286 [daq:INFO] Finished CT scan at 2024-09-09 07:32:12.286254, stop position : 200.0

        2024-09-09 07:32:18,240 [daq:INFO] Finished CT scan at 2024-09-09 07:32:18.240149, stop position : 200.0

        2024-09-09 07:32:30,842 [daq:INFO] Measure Dark field : 10 frames

        Transient Scan ID: 5632     Time: 2024-09-09 07:32:43
        Persistent Unique Scan ID: '9a5bfb26-1f22-4b3e-8ee2-4a1793bef789'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 07:32:45.0 |         1005.4 |
        |         2 | 07:32:46.8 |         1005.4 |
        |         3 | 07:32:48.6 |         1005.4 |
        |         4 | 07:32:50.5 |         1005.4 |
        |         5 | 07:32:52.3 |         1005.4 |
        |         6 | 07:32:54.1 |         1005.4 |
        |         7 | 07:32:55.9 |         1005.4 |
        |         8 | 07:32:57.7 |         1005.4 |
        |         9 | 07:32:59.5 |         1005.4 |
        |        10 | 07:33:01.3 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['9a5bfb26'] (scan num: 5632)



        2024-09-09 07:33:13,139 [daq:INFO] Measure Flat field : 10 frames


        Transient Scan ID: 5633     Time: 2024-09-09 07:33:44
        Persistent Unique Scan ID: '179db746-55e5-488f-8e8c-3e838b1f2472'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 07:33:46.6 |         1005.4 |
        |         2 | 07:33:48.4 |         1005.4 |
        |         3 | 07:33:50.2 |         1005.4 |
        |         4 | 07:33:52.0 |         1005.4 |
        |         5 | 07:33:53.8 |         1005.4 |
        |         6 | 07:33:55.6 |         1005.4 |
        |         7 | 07:33:57.4 |         1005.4 |
        |         8 | 07:33:59.2 |         1005.4 |
        |         9 | 07:34:01.0 |         1005.4 |
        |        10 | 07:34:02.8 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['179db746'] (scan num: 5633)





        Transient Scan ID: 5634     Time: 2024-09-09 07:35:02
        Persistent Unique Scan ID: 'd26ceaf0-a16f-4ac2-9854-0e6b7a1249b2'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 07:35:04.0 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['d26ceaf0'] (scan num: 5634)





        Transient Scan ID: 5635     Time: 2024-09-09 07:35:30
        Persistent Unique Scan ID: '0b5b29c2-69a3-40c5-a84d-6185d9b1d30b'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 07:35:32.5 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['0b5b29c2'] (scan num: 5635)





        Transient Scan ID: 5636     Time: 2024-09-09 07:35:59
        Persistent Unique Scan ID: '799acd23-c154-4a49-896d-b8bf4e470c1d'
        New stream: 'primary'
        +-----------+------------+----------------+
        |   seq_num |       time | pco_centroid_y |
        +-----------+------------+----------------+
        |         1 | 07:36:01.0 |         1005.4 |
        +-----------+------------+----------------+
        generator count ['799acd23'] (scan num: 5636)
