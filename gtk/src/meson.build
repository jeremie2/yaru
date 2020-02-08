gnome = import('gnome')
sassc = find_program('sassc')

# TODO rename variants to flavours
variants = []
foreach variant: ['default', 'dark', 'light']
  if not get_option(variant)
    message('skip variant ' + variant)
    continue
  endif
  variants += variant
endforeach

foreach variant: variants
  message('build variant ' + variant)

  if variant == 'default'
    theme_name = meson.project_name()
  else
    theme_name = meson.project_name() + '-' + variant
  endif

  theme_dir = join_paths(get_option('datadir'), 'themes', theme_name)


  # generate index.theme from template
  conf_data = configuration_data()
  conf_data.set('ThemeName', theme_name)
  conf_data.set('VariantThemeName', theme_name)
  configure_file(input: './common/index.theme.in',
    output: theme_name + '-index.theme',
    configuration: conf_data,
    install_dir: theme_dir)

  subdir(join_paths('flavours', variant, 'gtk-2.0'))

  foreach gtkver: ['3.0', '3.20']
    sources_relative_path = join_paths('flavours', variant, 'gtk-' + gtkver)

    gtk3_scss = run_command('cat',
      files(join_paths(sources_relative_path, 'meson-scss.list'))).stdout().split()
    gtk3_scss_dependencies = []
    foreach f: gtk3_scss
      gtk3_scss_dependencies += join_paths('flavours', variant, 'gtk-' + gtkver, f)
    endforeach

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

    assets_relative_path = run_command('cat',
      files(join_paths(sources_relative_path, 'meson-assets.list'))).stdout().split()
    gtk3_assets = []
    foreach f : assets_relative_path
      gtk3_assets += join_paths('flavours', variant, 'gtk-' + gtkver, f)
    endforeach

    install_data(gtk3_assets, install_dir: join_paths(theme_dir, 'gtk-' + gtkver, 'assets'))

  endforeach
endforeach

meson.add_install_script('post_install.py', get_option('datadir'), meson.project_name(), variants)
