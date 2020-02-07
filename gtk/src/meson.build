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

  subdir(join_paths('flavours', variant))

  foreach gtkver: ['3.20', '3.0']
    gtk3_scss_dependencies = []
    sources_relative_path = join_paths('flavours', variant, 'gtk-' + gtkver)
    gtk3_assets = run_command('cat', files(join_paths(sources_relative_path, 'meson-assets.list'))).stdout().split()

    generated_css = []
    foreach f : [ 'gtk', 'gtk-dark' ]
      gtk_sass_path = join_paths(sources_relative_path, f)
      # TODO review name generation (this doesn't seem really clear)
      target_name = '@0@-@1@-@2@-generated'.format(theme_name, f, gtkver)
      generated_css_out = '@0@-@1@-@2@-generated.css'.format(theme_name, f, gtkver)

      generated_css += custom_target(target_name,
        input: gtk_sass_path + '.scss',
        output: generated_css_out,
        depend_files: files(gtk3_scss_dependencies),
        command: [sassc, '-a', '@INPUT@', '@OUTPUT@'],
        install_dir: theme_dir
        )
    endforeach

    asset_xml = []
    foreach f : gtk3_assets
      asset_relative_path = join_paths('flavours', variant, 'gtk-' + gtkver, f)
      asset_xml += '<file>@0@</file>'.format(asset_relative_path)
    endforeach

    # gresource xml
    conf_data = configuration_data()
    conf_data.set('ASSETS', '\n'.join(asset_xml))
    conf_data.set('VARIANT', theme_name)
    conf_data.set('GTKVER', gtkver)

    gresource_relative_path = join_paths('common', 'gtk.gresource.xml.in')

    gresource_xml = configure_file(
      input: gresource_relative_path,
      output: theme_name + '-gtk-' + gtkver + '.gresource.xml',
      configuration: conf_data)

    gnome.compile_resources(theme_name + '-gtk-' + gtkver, gresource_xml,
      gresource_bundle: true,
      dependencies: generated_css,
      install: true,
      install_dir: join_paths(theme_dir, 'gtk-' + gtkver),
      )

    # css stubs
    conf_data = configuration_data()
    conf_data.set('VARIANT', theme_name)
    conf_data.set('GTKVER', gtkver)

    configure_file(input: './common/gtk.css.in',
      output: theme_name + '-gtk-' + gtkver + '.css',
      configuration: conf_data,
      install_dir: join_paths(theme_dir, 'gtk-' + gtkver)
      )

    configure_file(input: './common/gtk-dark.css.in',
      output: theme_name + '-gtk-' + gtkver + '-dark.css',
      configuration: conf_data,
      install_dir: join_paths(theme_dir, 'gtk-' + gtkver)
      )
  endforeach
endforeach

meson.add_install_script('post_install.py', get_option('datadir'), meson.project_name(), variants)
