yaml = require 'js-yaml'
path =  require 'path'
assets = require(path.resolve 'assets.yaml').shift()
_ = require('underscore')
{exec, spawn} = require 'child_process'
watch_r = require 'watch_r'
growl = require 'growl'
jade = require 'jade'
fs = require 'fs'



growl_success = (name) ->
  msg = "success: built #{name}"
  console.log msg
  growl msg, { image: 'tools/images/ok.png' }

growl_error = (name) ->
  msg = "error: failed to build #{name}"
  console.log msg
  growl msg, { image: 'tools/images/error.png' }


run = (arg_array, name) ->
    exec arg_array.join(' '), (err, stdout, stderr) ->
      if err
        growl_error name
      else
        growl_success name
      
      console.log stdout if stdout
      console.log stderr if stderr


task 'build:coffee', 'compiles all of the coffee-script source', () ->
  for section in assets.coffee.files
    coffee = ['coffee', '-c', '-l', '-o', assets.coffee.out_dir, '-j', section.output]
    
    #convert the input file names to their path
    input_files = _.map section.input, (fn) ->
      path.join assets.coffee.src_dir, fn
    
    coffee.push input_files.join(' ')
    run(coffee, 'coffee-script')

task 'build:less', 'compiles all of the less source', () ->
  for section in assets.less.files
    input = path.join assets.less.src_dir, section.input 
    output = path.join assets.less.out_dir, section.output 
    
    less = ['lessc', input, output]
    run(less, 'less')

task 'build:jade', 'compiles all of the jade source', () ->
  for section in assets.jade.files
    input = path.join assets.jade.src_dir, section.input 
    jade_command = ['jade', input, '--out', assets.jade.out_dir]
    run(jade_command , 'jade')

task 'build:templates', 'compiles all of the templates source', () ->
  output_string = ""
  namespaces = assets.templates.namespace.split('.')
  for n in [0..namespaces.length - 1]
    name = namespaces[0..n].join('.')
    line = "var #{name} = #{name} || {};"
    output_string  += line 
  
  for section in assets.templates.files
    input = path.join assets.templates.src_dir, section.input
    template_name = section.input.split('.')[0]
    output_string += "#{assets.templates.namespace}.#{template_name} = "
    
    data = fs.readFileSync(input , 'utf8')
    out = jade.compile data,
      filename: input
      client: true
      compileDebug: false
    output_string += out.toString()
  
  console.log output_string 

      
task 'watch', 'compiles all code on save', ->
  #watch coffee script
  if assets.coffee
    watch_r assets.coffee.src_dir, (err, watcher) ->
      watcher.on 'change', (target) ->
        console.log target.path
        growl "Building #{target.path}"
        invoke 'build:coffee'
  
  if assets.less
    watch_r assets.less.src_dir, (err, watcher) ->
      watcher.on 'change', (target) ->
        console.log target.path
        growl "Building #{target.path}"
        invoke 'build:less'
  
  if assets.jade
    watch_r assets.jade.src_dir, (err, watcher) ->
      watcher.on 'change', (target) ->
        console.log target.path
        growl "Building #{target.path}"
        invoke 'build:jade'
  
  if assets.templates
    watch_r assets.templates.src_dir, (err, watcher) ->
      watcher.on 'change', (target) ->
        console.log target.path
        growl "Building #{target.path}"
        invoke 'build:templates'
  
  
      
  