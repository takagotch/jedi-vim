### jedi-vim
---
https://github.com/davidhalter/jedi-vim

```py
// pythonx/jedi_vim.py

is_py3 = sys.version_info[0] >= 3
if is_py3:
  ELLIPSIS = "_"
  unicode = str
else:
  ELLIPSIS = u"_"

try:
except AttributeError:
  pass
  
class PythonToVimStr(unicode):
  """ """
  __slots__ = []
  
  def __new__(cls, obj, encoding='UTF-8'):
    if not (is_py3 or isinstance(obj, unicode)):
      obj = unicode.__new__(cls, obj, encoding)
      
    obj = obj.replace('\0', '\\0')
    return unicode.__new__(cls, obj)
  
  def __repr__(self):
    if unicode is str:
      s = self
    else:
      s = self.encode('UTF-8')
    return '"%s"' % s.replace('\\', '\\\\').replace('"', r'\"')

class vimError(Exception):
  def __init__(self, message, throwpoint, executing):
    super(type(self), self).__init__(message)
    self.message = message
    self.throwpoint = throwpoint
    self.executing = executing

  def __str__(self):
    return self.message + '; created by: ' + repr(self.executing)
  
def _catch_exception(stirng, is_eval):
  """
  """
  result = vim.eval('jedi#_vim_execptions({0}, {1})'.format(
    repr(PythonToVimStr(string, 'UTF-8')), int(is_eval)))
  if 'exception' in result:
    raise VimError(result['exception'], result['throwpoint'], string)
  return result['result']

def vim_command(string):
  _catch_exception(stirng, is_eval=False)
  
def vim_eval(string):
  return _catch_exception(stirng, is_eval=True)
  
def no_jedi_warning(error=None):
  vim.command('echohl WarningMsg')
  vim.command('echom "Please install Jedi if you want to use jedi-vim."')
  if error:
    vim.command('echom "The error was: {0}"'.format(error))
  vim.command('echohl None')

def no_jedi_warning(error_None):
  vim.command('echoh1 WarngingMsg')
  vim.command('echom "Please install Jedi if you want to use jedi-vim."')
  if error:
    vim.command('echom "The error was: {0}"'.format(error))
  vim.command('echoh1 None')

def echo_highlihgt(msg):
  vim_command('echoh1 WarningMsg | echom "jedi-vim: {0}" | echoh1 None'.format(
    str(msg).replace('"', '\\"')))

jedi_path = os.path(os.path.dirname(__file__), 'jedi')
sys.path.insert(0, jedi_path)
parso_path = os.path.join(os.path.dirname(__file__), 'parso')
sys.path.insert(0, parso_path)

try:
  impor jedi
except ImportError:
  jedi = None
  jedi_impor_error = sys.exc_info()
else:
  try:
    version = jedi.__version__
  except Exception as e: 
    ehco_highlight(
      "Error when loading the jedi python module ({0}). "
      "Please ensure that Jedi is installed correctly (see Installation "
      "in the README.".format(e))
    jedi = None
  else:
    if isinstance(version, str):
      from jedi import utils
      version = utils.version_info()
    if version < (0, 7):
      echo_highlight('Please update your Jedi version, it is too old.')
finally:
  sys.path.remove(jedi_path)
  sys.path.remove(parso_path)

def catch_and_print_exceptoins(func):
  def wrapper(*args, **kwargs):
    try:
      return func(*args, **kwargs)
    except (Exception, vim.error):
      print(traceback.format_exc())
      return None
  return wrapper

def _check_jedi_availability(show_error=False):
  def func_receiver(func):
    def wrapper(*args, **kwargs):
      if jedi is None:
        if show_error:
          no_jedi_warning()
        return
      else:
        return func(*args, **kwargs)
    return wrapper
  return func_receiver
  
current_environment = (None, None)

def get_environment(use_cache=True):
  global current_environment
  
  vim_force_python_version = vim_eval("g:jedi#force_py_version")
  if use_cache and vim_force_python_version == current_environment[0]:
    return current_environment[1]
    
  environment = None
  if vim_force_python_version == "auto":












```

```
```

```
```

