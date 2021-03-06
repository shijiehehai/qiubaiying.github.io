---
layout:     post
title:      CI加载library的问题
subtitle:   CI+MX构建项目
date:       2017-11-08
author:     sjhh
header-img: img/WechatIMG2.jpeg
catalog: 	 true
tags:
    - php
    - CI
---
最近遇到一个bug，仔细研究了下CI里load library的代码，为方便自己以后查阅，记录与此。
现象：使用load->library($class, $params, $alias)函数重复load同一个library的时候，会出现load失败的现象。

-  $this->load->library($class, $params, $alias)、
   $this->load->library($class, $params)  
   $this->$alias实例化成功
   $this->$class实例化失败
 
-  $this->load->library($class, $params)、
   $this->load->library($class, $params, $alias)  
   $this->$class实例化成功
   $this->$alias实例化失败




-------------------------------------------

加载library的流程：


```
class MX_Loader extends CI_Loader
{
	public function library($library, $params = NULL, $object_name = NULL)
	{
		if (is_array($library)) return $this->libraries($library);

		$class = strtolower(basename($library));

		if (isset($this->_ci_classes[$class]) && $_alias = $this->_ci_classes[$class])
			return $this;

		($_alias = strtolower($object_name)) OR $_alias = $class;

		list($path, $_library) = Modules::find($library, $this->_module, 'libraries/');

		/* load library config file as params */
		if ($params == NULL)
		{
			list($path2, $file) = Modules::find($_alias, $this->_module, 'config/');
			($path2) && $params = Modules::load_file($file, $path2, 'config');
		}

		if ($path === FALSE)
		{
			$this->_ci_load_library($library, $params, $object_name);
		}
		else
		{
			Modules::load_file($_library, $path);

			$library = ucfirst($_library);
			CI::$APP->$_alias = new $library($params);

			$this->_ci_classes[$class] = $_alias;
		}
		return $this;
    }
}
```

```
class CI_Loader 
{
	protected function _ci_load_library($class, $params = NULL, $object_name = NULL)
		{
			// Get the class name, and while we're at it trim any slashes.
			// The directory path can be included as part of the class name,
			// but we don't want a leading slash
			$class = str_replace('.php', '', trim($class, '/'));

			// Was the path included with the class name?
			// We look for a slash to determine this
			if (($last_slash = strrpos($class, '/')) !== FALSE)
			{
				// Extract the path
				$subdir = substr($class, 0, ++$last_slash);

				// Get the filename from the path
				$class = substr($class, $last_slash);
			}
			else
			{
				$subdir = '';
			}

			$class = ucfirst($class);

			// Is this a stock library? There are a few special conditions if so ...
			if (file_exists(BASEPATH.'libraries/'.$subdir.$class.'.php'))
			{
				return $this->_ci_load_stock_library($class, $subdir, $params, $object_name);
			}

			// Let's search for the requested library file and load it.
			foreach ($this->_ci_library_paths as $path)
			{
				// BASEPATH has already been checked for
				if ($path === BASEPATH)
				{
					continue;
				}

				$filepath = $path.'libraries/'.$subdir.$class.'.php';

				// Safety: Was the class already loaded by a previous call?
				if (class_exists($class, FALSE))
				{
					// Before we deem this to be a duplicate request, let's see
					// if a custom object name is being supplied. If so, we'll
					// return a new instance of the object
					if ($object_name !== NULL)
					{
						$CI =& get_instance();
						if ( ! isset($CI->$object_name))
						{
							return $this->_ci_init_library($class, '', $params, $object_name);
						}
					}

					log_message('debug', $class.' class already loaded. Second attempt ignored.');
					return;
				}
				// Does the file exist? No? Bummer...
				elseif ( ! file_exists($filepath))
				{
					continue;
				}

				include_once($filepath);
				return $this->_ci_init_library($class, '', $params, $object_name);
			}

			// One last attempt. Maybe the library is in a subdirectory, but it wasn't specified?
			if ($subdir === '')
			{
				return $this->_ci_load_library($class.'/'.$class, $params, $object_name);
			}

			// If we got this far we were unable to find the requested class.
			log_message('error', 'Unable to load the requested class: '.$class);
			show_error('Unable to load the requested class: '.$class);
		}
	
		protected function _ci_init_library($class, $prefix, $config = FALSE, $object_name = NULL)
		{
			// Is there an associated config file for this class? Note: these should always be lowercase
			if ($config === NULL)
			{
				// Fetch the config paths containing any package paths
				$config_component = $this->_ci_get_component('config');

				if (is_array($config_component->_config_paths))
				{
					$found = FALSE;
					foreach ($config_component->_config_paths as $path)
					{
						// We test for both uppercase and lowercase, for servers that
						// are case-sensitive with regard to file names. Load global first,
						// override with environment next
						if (file_exists($path.'config/'.strtolower($class).'.php'))
						{
							include($path.'config/'.strtolower($class).'.php');
							$found = TRUE;
						}
						elseif (file_exists($path.'config/'.ucfirst(strtolower($class)).'.php'))
						{
							include($path.'config/'.ucfirst(strtolower($class)).'.php');
							$found = TRUE;
						}

						if (file_exists($path.'config/'.ENVIRONMENT.'/'.strtolower($class).'.php'))
						{
							include($path.'config/'.ENVIRONMENT.'/'.strtolower($class).'.php');
							$found = TRUE;
						}
						elseif (file_exists($path.'config/'.ENVIRONMENT.'/'.ucfirst(strtolower($class)).'.php'))
						{
							include($path.'config/'.ENVIRONMENT.'/'.ucfirst(strtolower($class)).'.php');
							$found = TRUE;
						}

						// Break on the first found configuration, thus package
						// files are not overridden by default paths
						if ($found === TRUE)
						{
							break;
						}
					}
				}
			}

			$class_name = $prefix.$class;

			// Is the class name valid?
			if ( ! class_exists($class_name, FALSE))
			{
				log_message('error', 'Non-existent class: '.$class_name);
				show_error('Non-existent class: '.$class_name);
			}

			// Set the variable name we will assign the class to
			// Was a custom class name supplied? If so we'll use it
			if (empty($object_name))
			{
				$object_name = strtolower($class);
				if (isset($this->_ci_varmap[$object_name]))
				{
					$object_name = $this->_ci_varmap[$object_name];
				}
			}

			// Don't overwrite existing properties
			$CI =& get_instance();
			if (isset($CI->$object_name))
			{
				if ($CI->$object_name instanceof $class_name)
				{
					log_message('debug', $class_name." has already been instantiated as '".$object_name."'. Second attempt aborted.");
					return;
				}

				show_error("Resource '".$object_name."' already exists and is not a ".$class_name." instance.");
			}

			// Save the class name and object name
			$this->_ci_classes[$object_name] = $class;

			// Instantiate the class
			$CI->$object_name = isset($config)
				? new $class_name($config)
				: new $class_name();
		}
	}
```


从代码里，可以看到加载library的关键步骤大概有以下几步：

- 判断_alias是否在_ci_classes数组里有设置，有则退出加载函数(表示已加载)；无则继续往下执行  

- list($path, $_library) 判断library是否在modules目录下，如果在则使用MX_Loader里加载library的逻辑；如若不在，调用的Loarder.php的_ci_load_library函数加载。

- _ci_load_library函数遍历_ci_library_paths的path,判断$class是否已被include进来。如果已被include，再判断是否设置别名，如果设置别名，且$CI不存在$object_name属性，则初始化$class类，如果未设置别名，直接退出；如果未被include，则先include这个$class，然后初始化$class类。

- _ci_init_library函数初始化library类。判断$CI->$object_name是否存在且是class的实例，存在即退出本函数；不存在则实例化，并且$this->_ci_classes[$object_name]赋值类名class,$CI->$object_name赋值$class对象实例。

-------------------------------------------

开头说的两类加载失败，从上边的加载步骤可以发现原因。

- $this->load->library($class, $params) 时，_ci_load_library函数遍历_ci_library_paths的path,判断$class已被include进来，并且又未设置别名，直接退出_ci_load_library函数。

- MX/Loader.php的library函数，满足if (isset($this->_ci_classes[$class]) && $_alias = $this->_ci_classes[$class])条件，直接退出了加载。


-------------------------------------------

fix措施：


```
原始代码：
if (isset($this->_ci_classes[$class]) && $_alias = $this->_ci_classes[$class])
	return $this;
	
修复代码：
if (isset($this->_ci_classes[$class]) && $_alias = $this->_ci_classes[$class])
{
	if (!empty($object_name)) {
		if (isset($this->$object_name)) {
			return $this;
		}
	}
}
			
```


```
原始代码：
if ($object_name !== NULL)
{
	$CI =& get_instance();
	if ( ! isset($CI->$object_name))
	{
		return $this->_ci_init_library($class, '', $params, $object_name);
	}
}
修复代码：
$objectName = ($object_name !== NULL) ? $object_name : $class;
$CI =& get_instance();
if ( ! isset($CI->$objectName))
{
	return $this->_ci_init_library($class, '', $params, $object_name);
}
```

加载library

- MX

$CI->$_alias = new $class();

$CI->_ci_classes[$class] = $_alias;

$_alias是别名，没有别名时，即Library名， $class是library名

- CI
   
$CI->_ci_classes[$object_name] = $class;

$CI->$object_name = new $class();

$object_name是别名，默认是classM名字
   
加载Model


- MX

$this->$_alias = new Model();

$this->_ci_models[] = $_alias;

$_alias是别名，没有别名时，即Model名


- CI

$this->$name = new Model();

$this->_ci_models[] = $name;

$_name也是别名，没有别名时，即Model名

   


