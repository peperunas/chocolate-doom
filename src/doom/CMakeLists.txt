set(DOOM_SOURCES
    am_map.c
    deh_ammo.c
    deh_bexstr.c
    deh_cheat.c
    deh_doom.c
    deh_frame.c
    deh_misc.c
    deh_ptr.c
    deh_sound.c
    deh_thing.c
    deh_weapon.c
    d_items.c
    d_main.c
    d_net.c
    doomdef.c
    doomstat.c
    dstrings.c
    f_finale.c
    f_wipe.c
    g_game.c
    hu_lib.c
    hu_stuff.c
    info.c
    m_menu.c
    m_random.c
    p_ceilng.c
    p_doors.c
    p_enemy.c
    p_floor.c
    p_inter.c
    p_lights.c
    p_map.c
    p_maputl.c
    p_mobj.c
    p_plats.c
    p_pspr.c
    p_saveg.c
    p_setup.c
    p_sight.c
    p_spec.c
    p_switch.c
    p_telept.c
    p_tick.c
    p_user.c
    r_bsp.c
    r_data.c
    r_draw.c
    r_main.c
    r_plane.c
    r_segs.c
    r_sky.c
    r_things.c
    s_sound.c
    sounds.c
    statdump.c
    st_lib.c
    st_stuff.c
    wi_stuff.c)

# append -ropfuscated to source names
string(REGEX REPLACE "([\\w]*).c($|;)" "\\1-ropfuscated.c\\2" DOOM_ROP_SOURCES
                     "${DOOM_SOURCES}")
# append -ropfuscated to source names and using .s extension
string(REGEX REPLACE "([\\w]*).c($|;)" "\\1-ropfuscated.s\\2"
                     DOOM_ROP_ASM_SOURCES "${DOOM_SOURCES}")

set(DOOM_HEADERS
    am_map.h
    deh_misc.h
    d_englsh.h
    d_items.h
    d_main.h
    doomdata.h
    doomdef.h
    doomstat.h
    d_player.h
    dstrings.h
    d_textur.h
    d_think.h
    f_finale.h
    f_wipe.h
    g_game.h
    hu_lib.h
    hu_stuff.h
    info.h
    m_menu.h
    m_random.h
    p_inter.h
    p_local.h
    p_mobj.h
    p_pspr.h
    p_saveg.h
    p_setup.h
    p_spec.h
    p_tick.h
    r_bsp.h
    r_data.h
    r_defs.h
    r_draw.h
    r_local.h
    r_main.h
    r_plane.h
    r_segs.h
    r_sky.h
    r_state.h
    r_things.h
    s_sound.h
    sounds.h
    statdump.h
    st_lib.h
    st_stuff.h
    wi_stuff.h)

# append -ropfuscated to source names and using .s extension
set(DOOM_ABSPATH_HEADERS ${DOOM_HEADERS})
list(TRANSFORM DOOM_ABSPATH_HEADERS PREPEND ${CMAKE_CURRENT_SOURCE_DIR}/)

add_library(doom STATIC ${DOOM_SOURCES} ${DOOM_HEADERS})
add_library(doom-ropfuscated STATIC ${DOOM_ROP_ASM_SOURCES} ${DOOM_HEADERS})

# Get SDL's include directories
get_target_property(SDL_INCLUDES SDL2::SDL2 INCLUDE_DIRECTORIES)
get_target_property(SDLMIXER_INCLUDES SDL2::mixer INCLUDE_DIRECTORIES)
get_target_property(SDLNET_INCLUDES SDL2::net INCLUDE_DIRECTORIES)

# Construct the compiler string for the include directories.
foreach(dir ${SDL_INCLUDES} ${SDLMIXER_INCLUDES} ${SDLNET_INCLUDES})
  string(APPEND SDL_INCLUDES_STRING "-I${dir} ")
endforeach()

set(ROPF_IR_FLAGS
    "${ROPF_IR_FLAGS};-I${SDL2_INCLUDE_DIR};-I${PROJECT_BINARY_DIR};-I${PROJECT_SOURCE_DIR}/src;-I${PROJECT_SOURCE_DIR}/src/doom;${INCLUDES_DIRECTIVE}"
)

# obfuscate each source file
foreach(source ${DOOM_SOURCES})
  get_filename_component(source_name ${source} NAME_WE)

  generate_ropfuscated_asm(SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/${source} OUTNAME
                           ${source_name}-ropfuscated)
endforeach()

target_include_directories(doom PRIVATE "../"
                                        "${CMAKE_CURRENT_BINARY_DIR}/../../")
target_include_directories(doom-ropfuscated
                           PRIVATE "../" "${CMAKE_CURRENT_BINARY_DIR}/../../")
target_link_libraries(doom SDL2::SDL2 SDL2::mixer SDL2::net)
target_link_libraries(doom-ropfuscated SDL2::SDL2 SDL2::mixer SDL2::net)
