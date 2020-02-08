gnome = import('gnome')
sassc = find_program('sassc')

flavours = []
foreach flavour: ['default', 'dark', 'light']
  if not get_option(flavour)
    message('skip flavour ' + flavour)
    continue
  endif
  flavours += flavour
endforeach

foreach flavour: flavours
  message('build flavour ' + flavour)

  # default flavour is just Yaru
  if flavour == 'default'
    theme_name = meson.project_name()
  else
    theme_name = meson.project_name() + '-' + flavour
  endif

  theme_dir = join_paths(get_option('datadir'), 'themes', theme_name)

  conf_data = configuration_data()
  conf_data.set('ThemeName', theme_name)
  conf_data.set('FlavourThemeName', theme_name)
  configure_file(input: 'index.theme.in',
    output: theme_name + '-index.theme',
    configuration: conf_data,
    install_dir: theme_dir)

  # install gtk2 data
  subdir(join_paths('flavours', flavour, 'gtk-2.0'))

  # build and install gtk3 data
  foreach gtkver: ['3.0', '3.20']
    gtk3_assets = []
    gtk3_scss_dependencies = []

    # load gtk3 scss and assets from subdirs
    subdir(join_paths('flavours', flavour, 'gtk-' + gtkver))

    sources_relative_path = join_paths('flavours', flavour, 'gtk-' + gtkver)
    foreach f : [ 'gtk', 'gtk-dark' ]
      target_name = '@0@-@1@-@2@-generated'.format(theme_name, f, gtkver)
      gtk_sass_path = join_paths(sources_relative_path, f)

      custom_target(target_name,
        input: gtk_sass_path + '.scss',
        output: target_name + '.css',
        depend_files: files(gtk3_scss_dependencies),
        command: [sassc, '-a', '@INPUT@', '@OUTPUT@'],
        install: true,
        install_dir: join_paths(theme_dir, 'gtk-' + gtkver))
    endforeach

    install_data(gtk3_assets, install_dir: join_paths(theme_dir, 'gtk-' + gtkver, 'assets'))

  endforeach
endforeach

meson.add_install_script('post_install.py', get_option('datadir'), meson.project_name(), flavours)
