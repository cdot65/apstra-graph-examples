# Apstra Graph Explorer Examples

## Loopbacks

### Find loopback interface with a specific IP address

```python
match(
  node('system', name='device')
    .out('hosted_interfaces')
    .node('interface', name='intf', if_type='loopback')
    .where(lambda intf: intf.ipv4_addr == '10.0.0.2/32')
)
```

### Find associated loopbacks on all switches in a specified VRF

```python
match(
  node('security_zone', name='sz', label='blue')
    .out('instantiated_by')
    .node('sz_instance', name='sz_inst')
    .out('member_interfaces')
    .node('interface', if_type='loopback', name='loopback')
    .in_('hosted_interfaces')
    .node('system', role='leaf', name='system')
)
```

## Interfaces

### Find Systems with Interfaces connected as links

```python
node('system', name='system')
  .out('hosted_interfaces')
  .node('interface', name='interface')
  .out('link')
  .node('link', name='link')
```

### Find routed interfaces on spine switches

```python
node('system', role='spine', name='system')
  .out('hosted_interfaces')
  .node('interface', if_type='ip', name='interface')
```

### Count number of fabric links on spines

```python
match(
  node('system', role='spine', deploy_mode='deploy')
    .out('hosted_interfaces')
    .node('interface', name='leaf_intf')
    .out('link')
    .node('link', role='spine_leaf')
)
```

## Count number of fabric links on leafs

```python
match(
  node('system', role='leaf', deploy_mode='deploy')
    .out('hosted_interfaces')
    .node('interface', name='leaf_intf')
    .out('link')
    .node('link', role='spine_leaf')
    .in_('link')
    .node('interface')
    .in_('hosted_interfaces')
    .node('system', role='spine')
)
```

```python
match(
  node('system', role='leaf', deploy_mode='deploy', name='system')
    .out('hosted_interfaces')
    .node('interface', name='leaf_intf')
    .out('link')
    .node('link', role='spine_leaf')
)
```

## BGP

### Find all ASN within fabric

```python
match(
  node('domain', domain_type='autonomous_system', name='autonomous_system')
)
```

### Find all ASNs with Spine Loopbacks within the BGP peering domain

```python
match(
  node('domain', domain_type='autonomous_system', name='domain')
    .out('composed_of_systems')
    .node('system', role='spine', name='spine')
    .out('hosted_interfaces')
    .node('interface', name='router_loopback', if_type='loopback')
)
```

### Find ASN, Loopbacks, Systems, Security Zones, and SZ instances

```python
match(
  node('security_zone', name='sz')
    .out('instantiated_by')
    .node('sz_instance', name='sz_inst')
    .out('member_interfaces')
    .node('interface', if_type='loopback', name='loopback')
    .in_('hosted_interfaces')
    .node('system', role='leaf', name='system')
    .in_('composed_of_systems')
    .node('domain', domain_type='autonomous_system', name='domain'),
)
```

## security zone BGP peering query

```python
match(
  node('security_zone', name='sz')
    .out('instantiated_by')
    .node('sz_instance', name='sz_inst')
    .out('member_interfaces')
    .node('interface', if_type='loopback', name='loopback')
    .in_('hosted_interfaces')
    .node('system', role='leaf', name='system')
    .in_('composed_of_systems')
    .node('domain', domain_type='autonomous_system', name='domain'),
  node(name='sz_inst')
    .out('member_interfaces')
    .node('interface', if_type='subinterface', name='subinterface')
    .out('link')
    .node('link', name='link', link_type='logical_link')
    .in_('link')
    .node('interface', if_type='subinterface', name='remote_subinterface')
    .in_('composed_of')
    .node('interface', name='remote_interface')
    .in_('hosted_interfaces')
    .node('system', name='remote_system', role='external_router')
    .ensure_different('subinterface', 'remote_subinterface'),
  node(name='subinterface')
    .in_('composed_of')
    .node('interface', name='interface'),
  node('domain', domain_type='autonomous_system', name='remote_domain')
    .out('composed_of_systems')
    .node(name='remote_system')
    .out('hosted_interfaces')
    .node('interface', if_type='loopback', name='remote_loopback')
)
```

## Connected Interface for BGP query

```python
match(
    node('domain', name='domain', domain_type='autonomous_system',
           domain_id=not_none())
    .out('composed_of_systems')
    .node('system', role=ne('external_router'), name='border_switch')
    .out('hosted_interfaces')
    .node('interface', name='boarder_interface')
    .out('link')
    .node('link', name='link', role='to_external_router')
    .in_('link')
    .node('interface', name='router_interface')
    .in_('hosted_interfaces')
    .node('system', name='router', role='external_router')
    .in_('composed_of_systems')
    .node('domain', domain_type='autonomous_system',
          name='router_domain'),
    node('system', name='border_switch')
    .out('hosted_interfaces')
    .node('interface', if_type='loopback', name='boarder_loopback',
          loopback_id=0),
    node('system', name='router')
    .out('hosted_interfaces')
    .node('interface', if_type='loopback', name='router_loopback'))
```

## find rp for security zone

```python
match(
  node('security_zone', name='sz', label='blue')
    .in_('sz')
    .node('multicast_policy', name='mp')
    .out('rp')
    .node('rendezvous_point', name='rp')
)
```

## find all systems acting as RPs

```python
match(
  node('security_zone', name='sz', label='NCP')
  .in_('sz')
  .node('multicast_policy', name='mp')
  .out('rp')
  .node('rendezvous_point', name='rp')
  .out('hosted_on')
  .node('system', name='deivce')
)
```

## anycast peers

```python
match(
  node('security_zone', name='sz', label='NCP')
  .in_('sz')
  .node('multicast_policy', name='mp')
  .out('rp')
  .node('rendezvous_point', name='rp')
  .out('member_interfaces')
  .node('interface', name='peer')
)
```

## anycast interface(s)

```python
match(
  node('security_zone', name='sz', label='NCP')
  .in_('sz')
  .node('multicast_policy', name='mp')
  .out('rp')
  .node('rendezvous_point', name='rp')
  .out('anycast_interface')
  .node('interface', name='peer')
)
```

## Find all VTEPs

```python
match(
  node('interface', name='peer', if_type='logical_vtep')
)
```
