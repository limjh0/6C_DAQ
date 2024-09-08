Issues
======

``001`` .h5 file을 압축을 풀어 저장할 storage server의 target director가 없을 때 (없어졌을 때) 처리 (20240905)
--------------------------------------------------------------------------------------------------
아래와 같은 log가 나옴

::

   File "C:\codes\bl6c_daq\docker\tiff-writer\tiffWriterCallback.py", line 98, in resource
       os.makedirs(parent_dir)
       File "C:\Users\master\miniconda3\envs\bs_202302\lib\os.py", line 225, in makedirs
       mkdir(name, mode)
   FileExistsError: [WinError 183] 파일이 이미 있으므로 만들 수 없습니다: 'X:\\2024\\09\\05'
   Exception in callback Dispatcher.process(<DocumentNames.stop: 'stop'>, {'exit_status': 'success', 'num_events': {'primary': 10}, 'reason': '', 'run_start': 'b0a666e2-cd2...-65053e85091e', ...}) handle: <Handle Dispatcher.process(<DocumentNames.stop: 'stop'>, {'exit_status': 'success', 'num_events': {'primary': 10}, 'reason': '', 'run_start': 'b0a666e2-cd2...-65053e85091e', ...})>
   Traceback (most recent call last):
       File "C:\Users\master\miniconda3\envs\bs_202302\lib\asyncio\events.py", line 80, in _run
       self._context.run(self._callback, *self._args)


그러면서, storage server로 넘어가지 않고서 .h5 file이 ioc server에 남아있음

