Issues
======

``001`` .h5 file을 압축을 풀어 저장할 storage server의 target director가 없을 때 (없어졌을 때) 처리 (20240905)
--------------------------------------------------------------------------------------------------
아래와 같은 log가 나옴

::

    C:\codes\bl6c_daq\batch_scripts>C:\Windows\System32\cmd.exe /K "C:\Users\master\miniconda3\Scripts\activate.bat & conda activate bs_202302 & title tiffWriter & cd C:\codes\bl6c_daq\docker\tiff-writer & python tiffWriterCallback.py"
    2024-09-05 09:15:19,341 [save:INFO] source_file : Z:\Data\db\2024\09\05\76ed57e3-3d8b-4d93-b6cf_000000.h5
    Exception in callback Dispatcher.process(<DocumentName...e: 'resource'>, {'path_semantics': 'posix', 'resource_kwargs': {'frame_per_point': 1}, 'resource_path': '2024\\09\\05...cf_-000001.h5', 'root': 'Z:\\Data\\db', ...})
    handle: <Handle Dispatcher.process(<DocumentName...e: 'resource'>, {'path_semantics': 'posix', 'resource_kwargs': {'frame_per_point': 1}, 'resource_path': '2024\\09\\05...cf_-000001.h5', 'root': 'Z:\\Data\\db', ...})>
    Traceback (most recent call last):
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\asyncio\events.py", line 80, in _run
        self._context.run(self._callback, *self._args)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\run_engine.py", line 2461, in process
        exceptions = self.cb_registry.process(name, name.name, doc)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\utils\__init__.py", line 384, in process
        func(*args, **kwargs)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\utils\__init__.py", line 473, in __call__
        return mtd(*args, **kwargs)
    File "C:\codes\bl6c_daq\docker\tiff-writer\tiffWriterCallback.py", line 60, in __call__
        super().__call__(name, doc)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\event_model\__init__.py", line 123, in __call__
        return self._dispatch(name, doc, validate)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\event_model\__init__.py", line 131, in _dispatch
        output_doc = getattr(self, name)(doc)
    File "C:\codes\bl6c_daq\docker\tiff-writer\tiffWriterCallback.py", line 98, in resource
        os.makedirs(parent_dir)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\os.py", line 225, in makedirs
        mkdir(name, mode)
    FileExistsError: [WinError 183] 파일이 이미 있으므로 만들 수 없습니다: 'X:\\2024\\09\\05'
    Exception in callback Dispatcher.process(<DocumentNames.stop: 'stop'>, {'exit_status': 'success', 'num_events': {'primary': 10}, 'reason': '', 'run_start': '41c042a4-505...-a30592b7362f', ...})
    handle: <Handle Dispatcher.process(<DocumentNames.stop: 'stop'>, {'exit_status': 'success', 'num_events': {'primary': 10}, 'reason': '', 'run_start': '41c042a4-505...-a30592b7362f', ...})>
    Traceback (most recent call last):
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\asyncio\events.py", line 80, in _run
        self._context.run(self._callback, *self._args)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\run_engine.py", line 2461, in process
        exceptions = self.cb_registry.process(name, name.name, doc)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\utils\__init__.py", line 384, in process
        func(*args, **kwargs)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\utils\__init__.py", line 473, in __call__
        return mtd(*args, **kwargs)
    File "C:\codes\bl6c_daq\docker\tiff-writer\tiffWriterCallback.py", line 60, in __call__
        super().__call__(name, doc)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\event_model\__init__.py", line 123, in __call__
        return self._dispatch(name, doc, validate)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\event_model\__init__.py", line 131, in _dispatch
        output_doc = getattr(self, name)(doc)
    File "C:\codes\bl6c_daq\docker\tiff-writer\tiffWriterCallback.py", line 177, in stop
        for it in os.scandir(self.writepath):
    AttributeError: 'TiffWriterCallback' object has no attribute 'writepath'
    2024-09-05 09:15:49,050 [save:INFO] source_file : Z:\Data\db\2024\09\05\ea0b158b-8223-4b25-b7a0_000000.h5
    Exception in callback Dispatcher.process(<DocumentName...e: 'resource'>, {'path_semantics': 'posix', 'resource_kwargs': {'frame_per_point': 1}, 'resource_path': '2024\\09\\05...a0_-000001.h5', 'root': 'Z:\\Data\\db', ...})
    handle: <Handle Dispatcher.process(<DocumentName...e: 'resource'>, {'path_semantics': 'posix', 'resource_kwargs': {'frame_per_point': 1}, 'resource_path': '2024\\09\\05...a0_-000001.h5', 'root': 'Z:\\Data\\db', ...})>
    Traceback (most recent call last):
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\asyncio\events.py", line 80, in _run
        self._context.run(self._callback, *self._args)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\run_engine.py", line 2461, in process
        exceptions = self.cb_registry.process(name, name.name, doc)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\utils\__init__.py", line 384, in process
        func(*args, **kwargs)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\utils\__init__.py", line 473, in __call__
        return mtd(*args, **kwargs)
    File "C:\codes\bl6c_daq\docker\tiff-writer\tiffWriterCallback.py", line 60, in __call__
        super().__call__(name, doc)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\event_model\__init__.py", line 123, in __call__
        return self._dispatch(name, doc, validate)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\event_model\__init__.py", line 131, in _dispatch
        output_doc = getattr(self, name)(doc)
    File "C:\codes\bl6c_daq\docker\tiff-writer\tiffWriterCallback.py", line 98, in resource
        os.makedirs(parent_dir)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\os.py", line 225, in makedirs
        mkdir(name, mode)
    FileExistsError: [WinError 183] 파일이 이미 있으므로 만들 수 없습니다: 'X:\\2024\\09\\05'
    Exception in callback Dispatcher.process(<DocumentNames.stop: 'stop'>, {'exit_status': 'success', 'num_events': {'primary': 10}, 'reason': '', 'run_start': 'b0a666e2-cd2...-65053e85091e', ...})
    handle: <Handle Dispatcher.process(<DocumentNames.stop: 'stop'>, {'exit_status': 'success', 'num_events': {'primary': 10}, 'reason': '', 'run_start': 'b0a666e2-cd2...-65053e85091e', ...})>
    Traceback (most recent call last):
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\asyncio\events.py", line 80, in _run
        self._context.run(self._callback, *self._args)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\run_engine.py", line 2461, in process
        exceptions = self.cb_registry.process(name, name.name, doc)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\utils\__init__.py", line 384, in process
        func(*args, **kwargs)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\bluesky\utils\__init__.py", line 473, in __call__
        return mtd(*args, **kwargs)
    File "C:\codes\bl6c_daq\docker\tiff-writer\tiffWriterCallback.py", line 60, in __call__
        super().__call__(name, doc)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\event_model\__init__.py", line 123, in __call__
        return self._dispatch(name, doc, validate)
    File "C:\Users\master\miniconda3\envs\bs_202302\lib\site-packages\event_model\__init__.py", line 131, in _dispatch
        output_doc = getattr(self, name)(doc)
    File "C:\codes\bl6c_daq\docker\tiff-writer\tiffWriterCallback.py", line 195, in stop
        for it in os.scandir(self.writepath):
    AttributeError: 'TiffWriterCallback' object has no attribute 'writepath'
    2024-09-05 09:16:41,215 [save:INFO] source_file : Z:\Data\db\2024\09\05\abe33009-240c-4068-be16_000000.h5
    2024-09-05 09:17:18,053 [save:INFO] source_file : Z:\Data\db\2024\09\05\cc7d624d-03ec-4220-9ce0_000000.h5

그러면서, storage server로 넘어가지 않고서 .h5 file이 ioc server에 남아있음
