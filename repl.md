---
layout: default
title: Try it
permalink: /repl/
extra_footer: |
  <script type="text/javascript" src="/js/yaml.js"></script>
  <script type="text/javascript" src="/js/kaitai-struct-compiler-fastopt.js"></script>
  <script type="text/javascript" src="/js/repl.js"></script>
---
<link rel="stylesheet" href="{{ site.baseurl }}/styles/ks.css" type="text/css">
<link rel="stylesheet" href="{{ site.baseurl }}/styles/highlight/railscasts.css" type="text/css">

<div class="container">
        <div class="row">
            <div class="col-md-5"><h2>Kaitai Struct YAML</h2></div>
            <div class="col-md-7"><h2><select id="target_lang"></select></h2></div>
        </div>
        <div class="row">
            <div class="col-md-5"><textarea id="source" rows="30" style="width: 100%"></textarea></div>
            <div class="col-md-7">
                <pre><code class="java" id="compiled"></code></pre>
            </div>
        </div>

        <div style="background: red; color: white" id="err_msg"></div>
        <p><button class="btn btn-default" id="compile" onclick="ksCompile()">Compile!</button></p>
        <p>or try loading some examples: <span id="examples"></span></p>

        <div style="display: none">
<div class="example" data-name="DOS MZ">meta:
  id: dos_mz
  file-extension: exe
  endian: le
seq:
  - id: mz_header
    type: mz_header
  - id: mz_header2
    size: mz_header.relocations_ofs - 0x1c
  - id: relocations
    type: relocation
    repeat: expr
    repeat-expr: mz_header.qty_relocations
  - id: body
    size-eos: true
types:
  mz_header:
    seq:
      - id: magic
        size: 2
      - id: last_page_extra_bytes
        type: u2
      - id: qty_pages
        type: u2
      - id: qty_relocations
        type: u2
      - id: header_size
        type: u2
      - id: min_allocation
        type: u2
      - id: max_allocation
        type: u2
      - id: initial_ss
        type: u2
      - id: initial_sp
        type: u2
      - id: checksum
        type: u2
      - id: initial_ip
        type: u2
      - id: initial_cs
        type: u2
      - id: relocations_ofs
        type: u2
      - id: overlay_id
        type: u2
  relocation:
    seq:
      - id: ofs
        type: u2
      - id: seg
        type: u2
</div>
<div class="example" data-name="Doom .wad (simple)">meta:
  id: doom_wad
  endian: le
  file-extension: wad
  application: id Tech 1
seq:
  - id: magic
    type: str
    size: 4
    encoding: ASCII
  - id: index_qty
    type: s4
  - id: index_offset
    type: s4
instances:
  index:
    pos: index_offset
    type: index_entry
    repeat: expr
    repeat-expr: index_qty
types:
  index_entry:
    seq:
      - id: offset
        type: s4
      - id: size
        type: s4
      - id: name
        type: str
        size: 8
        encoding: ASCII
    instances:
      contents:
        io: _root._io
        pos: offset
        size: size
</div>
<div class="example" data-name="Doom .wad">meta:
  id: doom_wad
  endian: le
  file-extension: wad
  application: id Tech 1
seq:
  - id: magic
    type: str
    size: 4
    encoding: ASCII
  - id: qty_index_entries
    type: s4
  - id: index_offset
    type: s4
types:
  index_entry:
    seq:
      - id: offset
        type: s4
      - id: size
        type: s4
      - id: name
        type: str
        size: 8
        encoding: ASCII
    instances:
      contents:
        io: _root._io
        pos: offset
        size: size
        if: name != "THINGS\0\0" and name != "LINEDEFS"
      things_contents:
        io: _root._io
        pos: offset
        size: size
        if: name == "THINGS\0\0"
        type: thing
        repeat: eos
      linedef_contents:
        io: _root._io
        pos: offset
        size: size
        if: name == "LINEDEFS"
        type: linedef
        repeat: eos
  thing:
    seq:
      - id: x
        type: s2
      - id: y
        type: s2
      - id: angle
        type: u2
      - id: type
        type: u2
      - id: flags
        type: u2
  linedef:
    seq:
      - id: vertex_start_idx
        type: u2
      - id: vertex_end_idx
        type: u2
      - id: flags
        type: u2
      - id: line_type
        type: u2
      - id: sector_tag
        type: u2
      - id: sidedef_right_idx
        type: u2
      - id: sidedef_left_idx
        type: u2
instances:
  index:
    pos: index_offset
    type: index_entry
    repeat: expr
    repeat-expr: qty_index_entries
</div>
<div class="example" data-name="IPv4 packet">meta:
  id: ipv4_packet
seq:
  - id: b1
    type: u1
  - id: b2
    type: u1
  - id: total_length
    type: u2be
  - id: identification
    type: u2be
  - id: b67
    type: u2be
  - id: ttl
    type: u1
  - id: protocol
    type: u1
    enum: protocol
  - id: header_checksum
    type: u2be
  - id: src_ip_addr
    size: 4
  - id: dst_ip_addr
    size: 4
  - id: options
    size: ihl_bytes - 20
  - id: tcp_segment_body
    type: tcp_segment
    if: protocol == protocol::tcp
  - id: body
    size: total_length - 20
    if: protocol != protocol::tcp
enums:
  protocol:
    # http://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml
    0: hopopt
    1: icmp
    2: igmp
    3: ggp
    4: ipv4
    5: st
    6: tcp
    7: cbt
    8: egp
    9: igp
    10: bbn_rcc_mon
    11: nvp_ii
    12: pup
    13: argus
    14: emcon
    15: xnet
    16: chaos
    17: udp
    18: mux
    19: dcn_meas
    20: hmp
    21: prm
    22: xns_idp
    23: trunk_1
    24: trunk_2
    25: leaf_1
    26: leaf_2
    27: rdp
    28: irtp
    29: iso_tp4
    30: netblt
    31: mfe_nsp
    32: merit_inp
    33: dccp
    34: 3pc
    35: idpr
    36: xtp
    37: ddp
    38: idpr_cmtp
    39: tp_plus_plus
    40: il
    41: ipv6
    42: sdrp
    43: ipv6_route
    44: ipv6_frag
    45: idrp
    46: rsvp
    47: gre
    48: dsr
    49: bna
    50: esp
    51: ah
    52: i_nlsp
    53: swipe
    54: narp
    55: mobile
    56: tlsp
    57: skip
    58: ipv6_icmp
    59: ipv6_nonxt
    60: ipv6_opts
    61: any_host_internal_protocol
    62: cftp
    63: any_local_network
    64: sat_expak
    65: kryptolan
    66: rvd
    67: ippc
    68: any_distributed_file_system
    69: sat_mon
    70: visa
    71: ipcv
    72: cpnx
    73: cphb
    74: wsn
    75: pvp
    76: br_sat_mon
    77: sun_nd
    78: wb_mon
    79: wb_expak
    80: iso_ip
    81: vmtp
    82: secure_vmtp
    83: vines
    84: ttp
    84: iptm
    85: nsfnet_igp
    86: dgp
    87: tcf
    88: eigrp
    89: ospfigp
    90: sprite_rpc
    91: larp
    92: mtp
    93: ax_25
    94: ipip
    95: micp
    96: scc_sp
    97: etherip
    98: encap
    99: any_private_encryption_scheme
    100: gmtp
    101: ifmp
    102: pnni
    103: pim
    104: aris
    105: scps
    106: qnx
    107: a_n
    108: ipcomp
    109: snp
    110: compaq_peer
    111: ipx_in_ip
    112: vrrp
    113: pgm
    114: any_0_hop
    115: l2tp
    116: ddx
    117: iatp
    118: stp
    119: srp
    120: uti
    121: smp
    122: sm
    123: ptp
    124: isis_over_ipv4
    125: fire
    126: crtp
    127: crudp
    128: sscopmce
    129: iplt
    130: sps
    131: pipe
    132: sctp
    133: fc
    134: rsvp_e2e_ignore
    135: mobility_header
    136: udplite
    137: mpls_in_ip
    138: manet
    139: hip
    140: shim6
    141: wesp
    142: rohc
    255: reserved_255
instances:
  version:
    value: (b1 & 0xf0) >> 4
  ihl:
    value: b1 & 0xf
  ihl_bytes:
    value: ihl * 4
</div>
<div class="example" data-name="GIF image">meta:
  id: gif
  file-extension: gif
  endian: le
seq:
  - id: header
    type: header
  - id: logical_screen_descriptor
    type: logical_screen_descriptor
  - id: global_color_table
    type: global_color_table
    # https://www.w3.org/Graphics/GIF/spec-gif89a.txt - section 18
    if: '(logical_screen_descriptor.flags & 0x80) != 0'
    size: (2 << (logical_screen_descriptor.flags & 7)) * 3
  - id: blocks
    type: block
    repeat: eos
types:
  header:
    seq:
      - id: magic
        contents: 'GIF'
      - id: version
        size: 3
        encoding: ASCII
  logical_screen_descriptor:
    # https://www.w3.org/Graphics/GIF/spec-gif89a.txt - section 18
    seq:
      - id: screen_width
        type: u2
      - id: screen_height
        type: u2
      - id: flags
        type: u1
      - id: bg_color_index
        type: u1
      - id: pixel_aspect_ratio
        type: u1
  global_color_table:
    # https://www.w3.org/Graphics/GIF/spec-gif89a.txt - section 19
    seq:
      - id: entries
        type: color_table_entry
        repeat: eos
  color_table_entry:
    seq:
      - id: red
        type: u1
      - id: green
        type: u1
      - id: blue
        type: u1
  block:
    seq:
      - id: block_type
        type: u1
      - id: local_image_descriptor
        type: local_image_descriptor
        if: block_type == 0x2c
  local_image_descriptor:
    seq:
      - id: left
        type: u2
      - id: top
        type: u2
      - id: width
        type: u2
      - id: height
        type: u2
      - id: lid_flags
        type: u1
</div>
<div class="example" data-name="ZIP archive">meta:
  id: zip
  file-extension: zip
  endian: le
seq:
  - id: sections
    type: pk_section
    repeat: eos
types:
  pk_section:
    seq:
      - id: magic
        contents: 'PK'
      - id: section_type
        type: u2
      - id: central_dir_entry
        type: central_dir_entry
        if: section_type == 0x0201
      - id: local_file
        type: local_file
        if: section_type == 0x0403
      - id: end_of_central_dir
        type: end_of_central_dir
        if: section_type == 0x0605
  local_file:
    seq:
      - id: header
        type: local_file_header
      - id: body
        size: header.compressed_size
  local_file_header:
    seq:
      - id: version
        type: u2
      - id: flags
        type: u2
      - id: compression
        type: u2
        enum: compression
      - id: file_mod_time
        type: u2
      - id: file_mod_date
        type: u2
      - id: crc32
        type: u4
      - id: compressed_size
        type: u4
      - id: uncompressed_size
        type: u4
      - id: file_name_len
        type: u2
      - id: extra_len
        type: u2
      - id: file_name
        type: str
        size: file_name_len
        encoding: UTF-8
      - id: extra
        size: extra_len
  # https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT - 4.3.12
  central_dir_entry:
    seq:
      - id: version_made_by
        type: u2
      - id: version_needed_to_extract
        type: u2
      - id: flags
        type: u2
      - id: compression_method
        type: u2
      - id: last_mod_file_time
        type: u2
      - id: last_mod_file_date
        type: u2
      - id: crc32
        type: u4
      - id: compressed_size
        type: u4
      - id: uncompressed_size
        type: u4
      - id: file_name_len
        type: u2
      - id: extra_len
        type: u2
      - id: comment_len
        type: u2
      - id: disk_number_start
        type: u2
      - id: int_file_attr
        type: u2
      - id: ext_file_attr
        type: u4
      - id: local_header_offset
        type: s4
      - id: file_name
        type: str
        size: file_name_len
        encoding: UTF-8
      - id: extra
        size: extra_len
      - id: comment
        type: str
        size: comment_len
        encoding: UTF-8
  # https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT - 4.3.16
  end_of_central_dir:
    seq:
      - id: disk_of_end_of_central_dir
        type: u2
      - id: disk_of_central_dir
        type: u2
      - id: qty_central_dir_entries_on_disk
        type: u2
      - id: qty_central_dir_entries_total
        type: u2
      - id: central_dir_size
        type: u4
      - id: central_dir_offset
        type: u4
      - id: comment_len
        type: u2
      - id: comment
        type: str
        size: comment_len
        encoding: UTF-8
enums:
  compression:
    00: none
    01: shrunk
    02: reduced_1
    03: reduced_2
    04: reduced_3
    05: reduced_4
    06: imploded
    08: deflated
    09: enhanced_deflated
    10: pkware_dcl_imploded
    12: bzip2
    14: lzma
    18: ibm_terse
    19: ibm_lz77_z
    98: ppmd
</div>
</div>

</div>