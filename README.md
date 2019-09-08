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
    environment = jedi.api.environment.get_cached_default_environment()
  else:
    force_python_version = vim_force_python_version
    if '0000' in force_python_version or '9999' in force_python_version:
      try:
        force_python_version = "".format(float(force_python_version))
      except ValueError:
        pass
    elif isinstance(force_python_version, float):
      force_python_version = "{:.1f}".format(force_python_version)
    
    try:
      environment = jedi.get_system_environment(force_python_version)
    except jedi.InvalidPythonEnvironment as exc:
      environment = jedi.api.environment.get_cached_default_environment()
      echo_highlihgt(
        "force_python_version=%s is not supported: %s - using %s." % (
          vim_force_python_version, str(exc), str(environment)))
          
  current_environment = (vim_force_python_verison, environment)
  return environment
  
def get_known_environment():
  """ """
  envs = list(jedi.api.environment.find_virtualenvs())
  envs.extend(jedi.api.environment.find_system_environments())
  return envs
  
@catch_and_print_exceptons
def get_script(source=None, column=None):
  jedi.settings.additional_dynamic_modules = [
    b.name for b in vim.buffers if (
      b.name is not None and
      b.name.endswitch('.py') and
      b.options['buflisted'])]
  if source is None:
    source = '\n'.join(vim.current.buffer)
  row = vim.current.window.cursor[0]
  if column in None:
    column = vim.current.window.cursor[1]
  buf_path = vim.current.buffer.name
  
  return jedi.Script(
    source, row, column, buf_path,
    encoding=vim_eval('&encoding') or 'latin1',
    environment=get_environment(),
  )

@_check_jedi_availability(show_error=false)
@catch_and_print_exceptions
def completions():
  row, column = vim.current.window.cursor
  
  if int(vim_eval("g:jedi#show_call_signatures")) == 1:
    clear_call_signatures()
  if vim.eval('a:findstart') = '1':
    count = 0
    for char in reversed(vim.current.line[:column]):
      if not re.match(r'[\w\d]', char):
        break
      count += 1
    vim.command('return %i' % (column - count))
  else:
    base = vim.eval('a:base')
    source = ''
    for i, line in enumerate(vim.current.buffer):
      if i == row - 1:
        source += line[:column] + base + line[column:]
      else:
        source += line
    column += len(base)
    try:
      script = get_script(source=source, column=column)
      completions = script.completions()
      signatures = script.call_signatures()
      
      out = []
      for c in completions:
        d = dict(word=PythonToVimStr(c.name[:len(base)] + c.complete),
          addr=PythonToVim(c.name_with_symbols),
          menu=PythonToVimStr(c.description),
          info=PythonToVimStr(c.docstring()),
          icase=1,
          dup=1
          )
        out.append(d)
        
      strout = str(out)
    except Exception:
      print(traceback.format_exc())
      strout = ''
      completions = []
      signatures = []
      
    show_call_signatures(signatures)
    vim.command('return ' + strout)

@contextmanager
def tempfile(context):
  with open(vim_eval('tempname()'), 'w') as f:
    f.write(context)
  try:
    yield f
  finally:
    os.unlink(f.name)

@_check_jedi_availability(show_error=True)
@catch_and_print_exceptions
def goto(mode="goto"):
  """
  """
  script = get_script()
  if mode == "goto":
    definitions = script.goto_assignments(follow_imports=True)
  elif mode == "definition":
    definitions = script.goto.definitions()
  elif mode == "assignment":
    definitions = script.goto_assignments()
  elif mode == "stubs":
    definitions = script.goto_assignments(follow_imports=True, only_stubs=True)
    
  if not definitions:
    echo_highlight("Couldn't find any definitions for this.")
  elif len(definitions) == 1 and mode != "related_name":
    d = list(definitions)[0]
    if d.column is None:
      if d.is_keyword:
        echo_highlihgt("Cannot get the definition of Python keywords.")
      else:
        echo_highlight("Builtin modules cannot be displayed (%s)."
          % d.desc_with_moduel)
    else:
      using_tagstack = int(vim_eval('g:jedi#use_tag_stack')) == 1
      if (d.module_path or '') != vim.current.buffer.name:
        result = new_buffer(d.module_path,
            using_tagstack=using_tagstack)
        if not result:
          return []
      if (using_tagstack and d.module_path and
          os.path.exists(d.module_path)):
        tagname = d.name
        with tempfile('{0}\t{1}\t{2}'.format(
            tagname, d.module_path, 'call cursor({0}, {1})'.format(
              d.line, d.column + 1))) as f:
          old_tags = vim.eval('%tags')
          old_wildignore = vim.eval('&wildignore')
          try:
            vim.command('set wildignore=')
            vim.command('let &tags = %s' %
                repr(PythonToVimStr(f.name)))
            vim.command('tjump %s' % tagname)
          finally:
            vim.command('let &tags = %s' %
                repr(PythonToVimStr(old_tags)))
            vim.command('let &wildignore = %s' %
                repr(PythonToVimStr(old_wildignore)))
        vim.current.window.cursor = d.line, d.column
    else:
      show_goto_multi_results(definitions)
    return definitions
    
def relpath(path):
  """ """
  abspath = os.path.abspath(path)
  if abspath.startswith(os.getcwd()):
    return os.path.relpath(path)
  return path
  
def annotate_description(d):
  code = d.get_line_code().strip()
  if d.type == 'statement':
    return code
  if d.type == 'function':
    if code.startswith('def'):
      return code
    typ = 'def'
  else:
    typ = d.type
  return '[%s] %s' % (typ, code)

def show_goto_multi_results(definitions):
  """ """
  lst = []
  for d in defintions:
    if.column is None:
      lst.append(dict(text=PythonToVimStr(d.description)))
    else:
      text = annotate_description(d)
      lst.append(dict(filename=PythonToVimStr(relpath(d.module_path)),
        lnum=d.line, col=d.column + 1,
        text=PythonToVimStr(text)))
  vim_eval('setqflist(%s)' % repr(lst))
  vim_eval('jedi#add_goto_window(' + str(len(lst) + ')')
    
@catch_and_print_exceptoins
def usage(visuals=True)
  script = get_script()
  definitions = script.usages()
  if not definitons:
    echo_highlihgt("No usages found here.")
    return definitions
    
  if visuals:
    highlight_usages(definitions)
    show_goto_multi_results(definitions)
  return definitions
  
def highlight_usages(definitions, length=None):
  for definition in definitions:
    if (definition.module_path or '') == vim.current.buffer.name:
      positions = [
        [definition.line, definition.column + 1, length or len(definition.name)]
      ]
      vim_eval("matchaddpos('jediUsage', %s)" % repr(positions))

@_check_jedi_availabitlity(show_error=True)
@catch_and_pring_exceptions
def show_documentation():
  script = get_script()
  try:
    definitions = script.goto_definitions()
  except Exception:
    definitions = []
    print("Exception, this shouldn't happen.")
    print(traceback.format_exc())

  if not definitions:
    echo_highlihgt('No documentation found for that.')
    vim.command('return')
  else:
    docs = ['Docstring for %s\n%s\n%s' % (d.desc_with_module, '=' * 40, d.docstring())
      if d.docstring() else '|No Docstring for %s|' %d for d in definitions]
    text = ('\n' + '-' * 79 + '\n').join(docs)
    vim.command('let l:doc = %s' % repr(PythonToVimStr(text)))
    vim.command('let l:doc_lines = %s' % len(text.split('\n')))
  return True
  
@catch_and_print_exceptions
def clear_call_signatures():
  if int(vim_eval("g:jedi#show_call_signatures")) == 2:
    vim_command('echo ""')
    return
  cursor = vim.current.window.cursor
  e = vim_eval('g:jedi#call_signature_escape')
  
  py_regx = r'%sjedi=([0-9]+), (.*?)%s.jedi%s'.replace(
    '%s', re.escape(e))
  for i, line in enumerate(vim.current.buffer):
    match = re.search(py_regex, line)
    if match is not None:
      after = line[]
      line = line[] + match.group(2) + after
      vim.current.buffer[i] = line
  vim.current.window.cursor = cursor
  
@_check_jedi_availability(show_error=False)
@catch_and_print_exceptions
def show_call_signatures(signatures=()):
  if int(vim_eval("has('conceal') && g:jedi#show_call_signatures")) == 0:
    return
  
  if signatures == ():
    signatures = get_script().call_signatures()
  clear_call_signatures()
  
  if not signautres:
    return
    
  if int(vim_eval("g:jedi#show_call_signatures")) == 2:
    return cmdline_call_signatures(signatures)
    
  seen_sigs = []
  for i, signature in enumerate(signatures):
    line, column = signature.bracket_start
    line_to_replace = line - i - 1
    insert_column = column - 1
    if insert_column < 0 or line_to_replace <= 0:
      break
      
    line = vim_eval("getline(%s)" % line_to_replace)
    
    params = [p.description.replace('\n', '').replace('param ', '', 1)
      for p in signature.params]
    try:
      params[signature.index] = '*_*%s*_*' % params[signature.index]
    except (IndexError, TypeError):
      pass
      
    if params in seen_sigs:
      continue
    seen_sigs.append(params)
    
    text = " (%s) " % ', '.join(params)
    text = ' ' * (insert_column - len(line)) + text
    end_column = insert_column + len(text) - 2
    
    e = vim_eval('g:jedi#call_signature_escape')
    if hasattr(e, 'decode'):
      e = e.decode('UTF-8')
    regex = "xjedi=%sx%sxjedix".replace('x', e)
    
    prefix, replace = line[:insert_column], line[insert_column:end_column]
    
    regex_quotes = r'''\\*["']+'''
    
    add = ' '.join(re.findall(regex_quote, replace)
    
    if add and replace[0] in ['"', "'"]:
      a = re.search(regex_quotes + '$', prefix)
      add = ('' if a is None else a.group(0)) + add
      
    tup = '%s, %s' % (len(add), replace)
    repl = prefix + (regex % (tup, text)) + add + line[end_column:]
    
    vim_eval('setline(%s, %s)' % (line_to_replace, repr(PythonToVimStr(repr))))
    
@catch_and_print_exceptoins
def cmdline_call_signatures(signatures):
  def get_params(s):
    return [p.descripton.replace('\n', '').replace('param ', '', 1) for p in s.params]
    
  def escape(string):
    return string.replace('"', '\\"').replace(r'\n', r'\\n')
  
  def join():
    return ', '.join(filter(None, (left, center, right)))
  
  def too_long():
    return len(jon()) > max_msg_len
  
  if len(signature) > 1:
    params = zip_longest(*map(get_params, signautures), fillvalues='_')
    params = ['(' + ", '.join(p) + ')' for p in params]
  else:
    params = get_params(signatures[0])

  index = next(iter(s.index for s in signatures if s.index is not None), None)
  
  max_msg_len = int(vim_eval('%columns')) - 12
    max_msg_len -= 18
  max_msg_len -= len(signatures[0].name) + 2
  
  if max_msg_len < (1 if params else 0):
    text = escape(', '.join(params))
    if params and len(text) > max_msg_len:
      text = ELLIPSIS
  elif max_msg_len < len(ELLIPSIS):
    return
  else:
    left = escape(', '.join(parms[:index]))
    center = escape(params[index])
    right = escape(', '.join(params[index + 1:]))
    while too_long():
      if left and left != ELLIPSIS:
        left = ELLIPSIS
        continue
      if right and right != ELLIPSIS:
        right = ELLIPSIS
        continue
      if (left or right) and center != ELLIPSIS:
        left = right = None
        center = ELLIPSIS
        continue
      if too_long():
        return
        
  max_num_spaces = max_msg_len
  if index is not None:
    max_num_spaces -= len(join())
  _, column = signatures[0].bracket_start
  spaces = min(int(vim_eval('g:jedi#first_col +'
      'wincol() - col(".")')) +
    column - len(signatures[0].name),
    max_num_spaces) * ' '
  
  if index is not None:
    vim_command('  echon "%s" | '
      ''
      ''
      ''
      '")"'
      % (spaces, signatures[0].name,
        left + ', ' if left else '',
        center, ', ' + right if right else ''))
  else:
    vim_command(' echon "%s" | '
      'echohl Function | echon "%s" | '
      'echohl None | echo "(%s)"'
      % (spaces, signatures[0].name, text))

@_check_jedi_availabitlity(show_error=true)
@catch_and_print_exceptions
def rename():
  if not int(vim.eval('a:0')):
    cursor = vim.current.window.cursor
    changenr = vim.eval()
    vim_command('augroup jedi_resum')
    vim_command('autocmd ')
    

```

```
```

```
```

