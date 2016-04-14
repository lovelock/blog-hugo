+++
author = "frostwong"
date = "2016-04-09T21:55:00+08:00"
description = "通过编写扩展深入理解PHP"
draft = false
keywords = ["PHP", "扩展"]
tags = ["PHP", "扩展"]
title = "PHP扩展实战——用PHP实现类的原型"
type = "post"
isCJKLanguage = true

+++

在编写之前先用PHP实现这个类的原型吧。

```php
<?php

namespace Hylog;

use \DateTime;

class Hylog
{
    const HYLOG_VERSION = "0.1.0";

    const EMERGENCY = 'EMERGENCY';
    const ALERT     = 'ALERT';
    const CRITICAL  = 'CRITICAL';
    const ERROR     = 'ERROR';
    const WARNING   = 'WARNING';
    const NOTICE    = 'NOTICE';
    const INFO      = 'INFO';
    const DEBUG     = 'DEBUG';

    private static $_instance;

    private $_basePath;
    private $_sliceLogByHour;

    public function log($level, $message, $context = array())
    {
        $line = $this->interpolate($message, $context);

        $datetime         = new DateTime();
        $timestamp        = $datetime->getTimestamp();
        $formatedDatetime = $datetime->format(DateTime::ATOM);

        $line = $timestamp . "\t|\t" . $formatedDatetime . "\t|\t" . $line;

        $this->output($level, $line);
    }

    public function emergency($message, $context = array())
    {
        $this->log(self::EMERGENCY, $message, $context);
    }

    public function alert($message, $context = array())
    {
        $this->log(self::ALERT, $message, $context);
    }

    public function critical($message, $context = array())
    {
        $this->log(self::CRITICAL, $message, $context);
    }

    public function error($message, $context = array())
    {
        $this->log(self::ERROR, $message, $context);
    }

    public function warning($message, $context = array())
    {
        $this->log(self::WARING, $message, $context);
    }

    public function notice($message, $context = array())
    {
        $this->log(self::NOTICE, $message, $context);
    }

    public function info($message, $context = array())
    {
        $this->log(self::INFO, $message, $context);
    }

    public function debug($message, $context = array())
    {
        $this->log(self::DEBUG, $message, $context);
    }

    public static function getInstance() : object
    {
        if (!isset(self::$_instance)) {
            self::$_instance = new static();
        }

        return self::$_instance;
    }

    public function getVersion()
    {
        return self::HYLOG_VERSION;
    }

    public function setBasePath($path)
    {
        $this->_basePath = $path;
    }

    public function getBasePath() : string
    {
        return $this->_basePath;
    }

    public function setSliceByHour($bool)
    {
        $this->_sliceLogByHour = $bool;
    }

    public function getSliceByHour() : bool
    {
        return $this->_sliceLogByHour;
    }

    private function output($level, $message)
    {
        $logFile = $this->getLogFile($level);

        error_log($message . PHP_EOL, 3, $logFile);
    }

    private function getLogFile($level) : string
    {
        $cHour = date('ymdH');
        $cDay  = date('ymd');

        if ($this->_sliceLogByHour) {
            return $this->_basePath . '/' . $level . '.' . $cHour . '.log';
        } else {
            return $this->_basePath . '/' . $level . '.' . $cDay . '.log';
        }
    }

    private function interpolate($message, $context = array())
    {
        foreach ($context as $key => $val) {
            $replace['{' . $key . '}'] = $val;
        }

        return strtr($message, $replace);
    }

    private function __construct()
    {
        $this->_basePath = '/tmp/log';

        if (!is_dir($this->_basePath)) {
            mkdir($this->_basePath, 0700, true);
        } else {
            chmod($this->_basePath, 0700);
        }

        $this->_sliceLogByHour = true;
    }

    private function __clone()
    {
    }

    private function __wakeup()
    {
    }
}
```
