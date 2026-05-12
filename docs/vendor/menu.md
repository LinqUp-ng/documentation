# Menu Management API

This document provides the complete API implementation and integration guide for menu-related operations.

## API Implementation

```typescript
@/Users/ovieokomite-iffie/Documents/mobile apps/linqUp/api/handleMenu.ts:1-479
import { supabase, Tables } from '@/lib/supabase';
import { Response } from '@/types/api';
import { CompositeTypes, TablesInsert, TablesUpdate } from '@/types/database.types';

type MenuType = 'main' | 'side' | 'drinks' | 'snacks';

type MenuInsert = TablesInsert<'menus'>;
type MenuTypeInsert = TablesInsert<'menu_types'>;
type MenuGroupUpdate = TablesUpdate<'menu_groups'>;
type SubMenuInsert = TablesInsert<'sub_menus'>;
type MenuUpdate = TablesUpdate<'menus'>;
type MenuGroupInsert = TablesInsert<'menu_groups'>;
type SubMenuUpdate = TablesUpdate<'sub_menus'>;
type Menu = Tables<'menus'>;
type MenuGroup = Tables<'menu_groups'>;
type SubMenu = Tables<'sub_menus'>;

/**
 * Insert a new menu item
 * @param menuData - The menu data to insert
 * @returns The inserted menu item or error
 */
const insertMenu = async (
  menuData: MenuInsert
): Promise<Response<Menu>> => {
  try {
    const insertData: MenuInsert = {
      name: menuData.name,
      description: menuData.description,
      price: menuData.price,
      discount_price: menuData.discount_price,
      type: menuData.type || 'main',
      image_uri: menuData.image_uri,
      image_blur_hash: menuData.image_blur_hash,
      is_out_of_stock: menuData.is_out_of_stock ?? false,
      store_id: menuData.store_id,
    }; // Type assertion needed until database types are regenerated after migration

    const { data, error } = await supabase
      .from('menus')
      .insert(insertData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting menu:', error);
    throw error;
  }
};

/**
 * Update an existing menu item
 * @param menuId - The ID of the menu to update
 * @param menuData - The menu data to update
 * @returns The updated menu item or error
 */
const updateMenu = async (
  menuId: string,
  menuData: MenuUpdate
): Promise<Response<Menu>> => {
  try {
    const updateData: MenuUpdate = {
      name: menuData.name,
      description: menuData.description,
      price: menuData.price,
      discount_price: menuData.discount_price,
      type: menuData.type,
      image_uri: menuData.image_uri,
      image_blur_hash: menuData.image_blur_hash,
      is_out_of_stock: menuData.is_out_of_stock,
    }; // Type assertion needed until database types are regenerated after migration

    const { data, error } = await supabase
      .from('menus')
      .update(updateData)
      .eq('id', menuId)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item updated successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error updating menu:', error);
    throw error;
  }
};

/**
 * Soft delete a menu item by calling the database function
 * This marks the menu as deleted and changes store_id to a placeholder UUID
 * @param menuId - The ID of the menu to delete
 * @returns Success or error
 */
const deleteMenu = async (menuId: string): Promise<Response<void>> => {
    try {
        const { error } = await supabase.rpc('delete_menu', {
        p_menu_id: menuId,
        }); // Type assertion needed until database types are regenerated after migration

        if (error) throw error;

        return {
        isSuccessful: true,
        message: 'Menu item deleted successfully',
        data: undefined,
        };
    } catch (error) {
        console.error('Error deleting menu:', error);
        throw error;
    }
};

/**
 * Insert a new menu type
 * @param menuTypeData - The menu type data to insert
 * @returns The inserted menu type or error
 */
const addMenuType = async (
  menuTypeData: MenuTypeInsert
): Promise<Response<Tables<'menu_types'>>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .insert(menuTypeData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu type inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting menu type:', error);
    throw error;
  }
};

/**
 * Get a menu type by name
 * @param name - The name of the menu type
 * @returns The menu type or error
 */
const getMenuTypeByName = async (
  name: string
): Promise<Response<Tables<'menu_types'>>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .select()
      .eq('name', name)
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu type retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu type by name:', error);
    throw error;
  }
};

/**
 * Get a menu type by ID
 * @param id - The ID of the menu type
 * @returns The menu type or error
 */
const getMenuTypeById = async (
  id: string
): Promise<Response<Tables<'menu_types'>>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .select()
      .eq('id', id)
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu type retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu type by ID:', error);
    throw error;
  }
};

/**
 * Get all menu types
 * @returns All menu types or error
 */
const getAllMenuTypes = async (): Promise<Response<Tables<'menu_types'>[]>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .select()
      .order('created_at', { ascending: false });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu types retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu types:', error);
    throw error;
  }
};

/**
 * Insert a new menu group
 * @param menuGroupData - The menu group data to insert
 * @returns The inserted menu group or error
 */
const addMenuGroup = async (
  menuGroupData: MenuGroupInsert
): Promise<Response<MenuGroup>> => {
  try {
    const { data, error } = await supabase
      .from('menu_groups')
      .insert(menuGroupData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu group inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting menu group:', error);
    throw error;
  }
};

/**
 * Update an existing menu group
 * @param menuGroupId - The ID of the menu group to update
 * @param menuGroupData - The menu group data to update
 * @returns The updated menu group or error
 */
const updateMenuGroup = async (
  menuGroupId: string,
  menuGroupData: MenuGroupUpdate
): Promise<Response<MenuGroup>> => {
  try {
    const { data, error } = await supabase
      .from('menu_groups')
      .update(menuGroupData)
      .eq('id', menuGroupId)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu group updated successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error updating menu group:', error);
    throw error;
  }
};

/**
 * Delete a menu group
 * @param menuGroupId - The ID of the menu group to delete
 * @returns Success or error
 */
const deleteMenuGroup = async (
  menuGroupId: string
): Promise<Response<void>> => {
  try {
    const { error } = await supabase
      .from('menu_groups')
      .delete()
      .eq('id', menuGroupId);

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu group deleted successfully',
      data: undefined,
    };
  } catch (error) {
    console.error('Error deleting menu group:', error);
    throw error;
  }
};

/**
 * Insert a new sub menu
 * @param subMenuData - The sub menu data to insert
 * @returns The inserted sub menu or error
 */
const insertSubMenu = async (
  subMenuData: SubMenuInsert
): Promise<Response<SubMenu>> => {
  try {
    const { data, error } = await supabase
      .from('sub_menus')
      .insert(subMenuData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menu inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting sub menu:', error);
    throw error;
  }
};

/**
 * Insert multiple sub menus in one shot
 * @param subMenuDataArray - Array of sub menu data to insert
 * @returns The inserted sub menus or error
 */
const insertSubMenus = async (
  subMenuDataArray: SubMenuInsert[]
): Promise<Response<SubMenu[]>> => {
  try {
    const { data, error } = await supabase
      .from('sub_menus')
      .insert(subMenuDataArray)
      .select();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menus inserted successfully',
      data: data || [],
    };
  } catch (error) {
    console.error('Error inserting sub menus:', error);
    throw error;
  }
};

/**
 * Update an existing sub menu
 * @param subMenuId - The ID of the sub menu to update
 * @param subMenuData - The sub menu data to update
 * @returns The updated sub menu or error
 */
const updateSubMenu = async (
  subMenuId: string,
  subMenuData: SubMenuUpdate
): Promise<Response<SubMenu>> => {
  try {
    const { data, error } = await supabase
      .from('sub_menus')
      .update(subMenuData)
      .eq('id', subMenuId)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menu updated successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error updating sub menu:', error);
    throw error;
  }
};

/**
 * Delete a sub menu
 * @param subMenuId - The ID of the sub menu to delete
 * @returns Success or error
 */
const deleteSubMenu = async (
  subMenuId: string
): Promise<Response<void>> => {
  try {
    const { error } = await supabase
      .from('sub_menus')
      .delete()
      .eq('id', subMenuId);

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menu deleted successfully',
      data: undefined,
    };
  } catch (error) {
    console.error('Error deleting sub menu:', error);
    throw error;
  }
};

/**
 * Get menus by store ID with cursor pagination
 * @param storeId - The store ID
 * @param cursorId - The cursor ID for pagination (optional)
 * @param cursorCreatedAt - The cursor created_at timestamp for pagination (optional)
 * @param limit - The number of records to return (default 20)
 * @returns The menus or error
 */
const getMenuByStoreID = async (
  storeId: string,
  cursorId?: string | null,
  cursorCreatedAt?: string | null,
  limit = 20
): Promise<Response<Menu[]>> => {
    try {
        const { data, error } = await supabase.rpc('get_menu_by_store_id', {
            p_store_id: storeId,
            p_limit: limit,
            ...(cursorId && { p_cursor_id: cursorId }),
            ...(cursorCreatedAt && { p_cursor_created_at: cursorCreatedAt }),
        });

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Menus retrieved successfully',
            data: data,
        };
    } catch (error) {
        console.error('Error retrieving menus by store ID:', error);
        throw error;
    }
};

/**
 * Get a single menu item with nested menu_groups and sub_menus
 * @param menuId - The menu ID
 * @returns The menu item with nested data or error
 */
const getMenuItem = async (
  menuId: string
): Promise<Response<CompositeTypes<'menu_item_response'>>> => {
  try {
    const { data, error } = await supabase.rpc('get_menu_item', {
      p_menu_id: menuId,
    });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu item:', error);
    throw error;
  }
};

export default {
    insertMenu,
    updateMenu,
    deleteMenu,
    addMenuType,
    getMenuTypeByName,
    getMenuTypeById,
    getAllMenuTypes,
    addMenuGroup,
    updateMenuGroup,
    deleteMenuGroup,
    insertSubMenu,
    insertSubMenus,
    updateSubMenu,
    deleteSubMenu,
    getMenuByStoreID,
    getMenuItem,
};
```

## Functions

### `insertMenu`
Inserts a new menu item into the database.

**Important:** The `type` parameter must be a valid entry from the `menu_types` table. You should first retrieve all menu types using `getAllMenuTypes()` and use the `name` field from the desired menu type as the `type` parameter.

```typescript
const insertMenu = async (
  menuData: MenuInsert
): Promise<Response<Menu>> => {
  try {
    const insertData: MenuInsert = {
      name: menuData.name,
      description: menuData.description,
      price: menuData.price,
      discount_price: menuData.discount_price,
      type: menuData.type || 'main',
      image_uri: menuData.image_uri,
      image_blur_hash: menuData.image_blur_hash,
      is_out_of_stock: menuData.is_out_of_stock ?? false,
      store_id: menuData.store_id,
    };

    const { data, error } = await supabase
      .from('menus')
      .insert(insertData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting menu:', error);
    throw error;
  }
};
```

### `updateMenu`
Updates an existing menu item.

```typescript
const updateMenu = async (
  menuId: string,
  menuData: MenuUpdate
): Promise<Response<Menu>> => {
  try {
    const updateData: MenuUpdate = {
      name: menuData.name,
      description: menuData.description,
      price: menuData.price,
      discount_price: menuData.discount_price,
      type: menuData.type,
      image_uri: menuData.image_uri,
      image_blur_hash: menuData.image_blur_hash,
      is_out_of_stock: menuData.is_out_of_stock,
    };

    const { data, error } = await supabase
      .from('menus')
      .update(updateData)
      .eq('id', menuId)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item updated successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error updating menu:', error);
    throw error;
  }
};
```

### `deleteMenu`
Soft deletes a menu item using a database function.

```typescript
const deleteMenu = async (menuId: string): Promise<Response<void>> => {
  try {
    const { error } = await supabase.rpc('delete_menu', {
      p_menu_id: menuId,
    });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item deleted successfully',
      data: undefined,
    };
  } catch (error) {
    console.error('Error deleting menu:', error);
    throw error;
  }
};
```

### `addMenuType`
Adds a new menu category type.

```typescript
const addMenuType = async (
  menuTypeData: MenuTypeInsert
): Promise<Response<Tables<'menu_types'>>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .insert(menuTypeData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu type inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting menu type:', error);
    throw error;
  }
};
```

### `getMenuTypeByName`
Retrieves a menu type definition by its name.

```typescript
const getMenuTypeByName = async (
  name: string
): Promise<Response<Tables<'menu_types'>>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .select()
      .eq('name', name)
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu type retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu type by name:', error);
    throw error;
  }
};
```

### `getMenuTypeById`
Retrieves a menu type definition by its ID.

```typescript
const getMenuTypeById = async (
  id: string
): Promise<Response<Tables<'menu_types'>>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .select()
      .eq('id', id)
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu type retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu type by ID:', error);
    throw error;
  }
};
```

### `getAllMenuTypes`
Retrieves all available menu types.

```typescript
const getAllMenuTypes = async (): Promise<Response<Tables<'menu_types'>[]>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .select()
      .order('created_at', { ascending: false });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu types retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu types:', error);
    throw error;
  }
};
```

### `addMenuGroup`
Adds a new group (category of options) to a menu.

```typescript
const addMenuGroup = async (
  menuGroupData: MenuGroupInsert
): Promise<Response<MenuGroup>> => {
  try {
    const { data, error } = await supabase
      .from('menu_groups')
      .insert(menuGroupData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu group inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting menu group:', error);
    throw error;
  }
};
```

### `updateMenuGroup`
Updates an existing menu group.

```typescript
const updateMenuGroup = async (
  menuGroupId: string,
  menuGroupData: MenuGroupUpdate
): Promise<Response<MenuGroup>> => {
  try {
    const { data, error } = await supabase
      .from('menu_groups')
      .update(menuGroupData)
      .eq('id', menuGroupId)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu group updated successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error updating menu group:', error);
    throw error;
  }
};
```

### `deleteMenuGroup`
Deletes a menu group.

```typescript
const deleteMenuGroup = async (
  menuGroupId: string
): Promise<Response<void>> => {
  try {
    const { error } = await supabase
      .from('menu_groups')
      .delete()
      .eq('id', menuGroupId);

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu group deleted successfully',
      data: undefined,
    };
  } catch (error) {
    console.error('Error deleting menu group:', error);
    throw error;
  }
};
```

### `insertSubMenu`
Adds a specific sub-menu item (option) to a group.

```typescript
const insertSubMenu = async (
  subMenuData: SubMenuInsert
): Promise<Response<SubMenu>> => {
  try {
    const { data, error } = await supabase
      .from('sub_menus')
      .insert(subMenuData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menu inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting sub menu:', error);
    throw error;
  }
};
```

### `insertSubMenus`
Inserts multiple sub-menu items (options) to a group in a single batch operation.

```typescript
const insertSubMenus = async (
  subMenuDataArray: SubMenuInsert[]
): Promise<Response<SubMenu[]>> => {
  try {
    const { data, error } = await supabase
      .from('sub_menus')
      .insert(subMenuDataArray)
      .select();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menus inserted successfully',
      data: data || [],
    };
  } catch (error) {
    console.error('Error inserting sub menus:', error);
    throw error;
  }
};
```

### `updateSubMenu`
Updates an existing sub-menu item.

```typescript
const updateSubMenu = async (
  subMenuId: string,
  subMenuData: SubMenuUpdate
): Promise<Response<SubMenu>> => {
  try {
    const { data, error } = await supabase
      .from('sub_menus')
      .update(subMenuData)
      .eq('id', subMenuId)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menu updated successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error updating sub menu:', error);
    throw error;
  }
};
```

### `deleteSubMenu`
Deletes a sub-menu item.

```typescript
const deleteSubMenu = async (
  subMenuId: string
): Promise<Response<void>> => {
  try {
    const { error } = await supabase
      .from('sub_menus')
      .delete()
      .eq('id', subMenuId);

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menu deleted successfully',
      data: undefined,
    };
  } catch (error) {
    console.error('Error deleting sub menu:', error);
    throw error;
  }
};
```

### `getMenuByStoreID`
Retrieves a paginated list of menus for a specific store.

```typescript
const getMenuByStoreID = async (
  storeId: string,
  cursorId?: string | null,
  cursorCreatedAt?: string | null,
  limit = 20
): Promise<Response<Menu[]>> => {
    try {
        const { data, error } = await supabase.rpc('get_menu_by_store_id', {
            p_store_id: storeId,
            p_limit: limit,
            ...(cursorId && { p_cursor_id: cursorId }),
            ...(cursorCreatedAt && { p_cursor_created_at: cursorCreatedAt }),
        });

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Menus retrieved successfully',
            data: data,
        };
    } catch (error) {
        console.error('Error retrieving menus by store ID:', error);
        throw error;
    }
};
```

### `getMenuItem`
Retrieves a single menu item with fully nested menu groups and sub-menus.

```typescript
const getMenuItem = async (
  menuId: string
): Promise<Response<CompositeTypes<'menu_item_response'>>> => {
  try {
    const { data, error } = await supabase.rpc('get_menu_item', {
      p_menu_id: menuId,
    });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu item:', error);
    throw error;
  }
};
```

## Integration Examples

### Step 1: Upload Image (Required)
Before adding a menu item, you **must** upload its image to generate a storage URI and a blurhash. This ensures the app can display a beautiful placeholder while the high-resolution image loads.

Use the `uploadWithBlurhash` function from `handleUpload.ts`:

```typescript
import uploadApi from '@/api/handleUpload';

// id: current user or store ID
// uri: local file path from image picker
const uploadResult = await uploadApi.uploadWithBlurhash({
  id: storeId,
  uri: selectedImageUri,
  mimeType: 'image/jpeg',
  bucketName: 'menus',
  fileName: 'signature-dish'
});

if (uploadResult.isSuccessful) {
  const { uri, blur_hash } = uploadResult;
  // Proceed to Step 2
}
```

### Step 2: Add Menu Item
Pass the `uri` and `blur_hash` from Step 1 into the `insertMenu` function.

**Important:** The `type` parameter must be a valid entry from the `menu_types` table. First retrieve all menu types, then use the `name` field from the desired menu type.

```typescript
import menuApi from '@/api/handleMenu';

// Step 2a: Get available menu types
const menuTypesResult = await menuApi.getAllMenuTypes();

if (!menuTypesResult.isSuccessful) {
  throw new Error('Failed to retrieve menu types');
}

// Step 2b: Select the desired menu type by name
const menuType = menuTypesResult.data.find(mt => mt.name === 'Main');
if (!menuType) {
  throw new Error('Menu type not found');
}

// Step 2c: Insert menu item with the selected type
const menuResult = await menuApi.insertMenu({
  name: "Signature Burger",
  description: "Our world-famous beef burger",
  price: 15000,
  image_uri: uploadResult.uri, // From Step 1
  image_blur_hash: uploadResult.blur_hash, // From Step 1
  store_id: storeId,
  type: menuType.name // Use the name from the menu_types table
});
```

### Get Menus by Store
```typescript
import menuApi from '@/api/handleMenu';

const result = await menuApi.getMenuByStoreID(storeId, null, null, 20);

if (result.isSuccessful) {
  const menus = result.data;
  // Use menus...
}
```

### Get Single Menu Item (Nested)
Retrieves a single menu item with all its associated groups and sub-menus.

```typescript
import menuApi from '@/api/handleMenu';

const result = await menuApi.getMenuItem(menuId);

if (result.isSuccessful) {
  const menu = result.data;
  console.log(menu.name);
  console.log(menu.menu_groups); // Array of groups with sub_menus
}
```

### Add Menu Groups and Sub-Menus
To add sub-menu options to a menu item, you must first create a menu group, then add sub-menus that reference that group.

#### Step 1: Create a Menu Group
A menu group represents a category of options (e.g., "Size", "Toppings", "Sides").

```typescript
import menuApi from '@/api/handleMenu';

const groupResult = await menuApi.addMenuGroup({
  menu_id: menuId, // The menu item this group belongs to
  food_category: "Size",
  max_item_count: 1, // Maximum number of options user can select
  is_required: true, // Whether this group is mandatory
});

if (groupResult.isSuccessful) {
  const groupId = groupResult.data.id;
  // Proceed to Step 2
}
```

#### Step 2: Add Sub-Menus to the Group
Sub-menus are the individual options within a group (e.g., "Small", "Medium", "Large").

**Option A: Insert one at a time**

```typescript
import menuApi from '@/api/handleMenu';

// Add multiple sub-menu options to the group
const subMenu1 = await menuApi.insertSubMenu({
  menu_group_id: groupId, // Reference to the group created in Step 1
  name: "Small",
  price: 5000, // Additional cost (can be negative for discounts)
});

const subMenu2 = await menuApi.insertSubMenu({
  menu_group_id: groupId,
  name: "Medium",
  price: 2000, // Additional 2000 currency units
});

const subMenu3 = await menuApi.insertSubMenu({
  menu_group_id: groupId,
  name: "Large",
  price: 4000, // Additional 4000 currency units
});
```

**Option B: Insert multiple in one shot (recommended for bulk operations)**

```typescript
import menuApi from '@/api/handleMenu';

// Add multiple sub-menu options in a single batch operation
const subMenus = await menuApi.insertSubMenus([
  {
    menu_group_id: groupId,
    name: "Small",
    price: 5000,
  },
  {
    menu_group_id: groupId,
    name: "Medium",
    price: 2000,
  },
  {
    menu_group_id: groupId,
    name: "Large",
    price: 4000,
  }
]);

if (subMenus.isSuccessful) {
  console.log('Inserted sub-menus:', subMenus.data);
}
```

#### Complete Example: Adding Multiple Groups with Options

```typescript
import menuApi from '@/api/handleMenu';

// Create a "Size" group
const sizeGroup = await menuApi.addMenuGroup({
  menu_id: menuId,
  food_category: "Size",
  max_item_count: 1, // Maximum number of options user can select
  is_required: true, // Whether this group is mandatory
});

// Add size options
await menuApi.insertSubMenu({
  menu_group_id: sizeGroup.data.id,
  name: "Small",
  price: 4000,
});

await menuApi.insertSubMenu({
  menu_group_id: sizeGroup.data.id,
  name: "Medium",
  price: 2000,
});

// Create a "Toppings" group (multiple selections allowed)
const toppingsGroup = await menuApi.addMenuGroup({
  menu_id: menuId,
  food_category: "Toppings",
  max_item_count: 5, // Maximum number of options user can select
  is_required: false, // Whether this group is mandatory
});

// Add topping options
await menuApi.insertSubMenu({
  menu_group_id: toppingsGroup.data.id,
  name: "Cheese",
  price: 500,
});

await menuApi.insertSubMenu({
  menu_group_id: toppingsGroup.data.id,
  name: "Bacon",
  price: 1000,
});
```
