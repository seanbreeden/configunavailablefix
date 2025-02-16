      ______                                                              
     /      \                                                             
    |  $$$$$$\  ______    ______   _______                                
    | $$___\$$ /      \  |      \ |       \                               
     \$$    \ |  $$$$$$\  \$$$$$$\| $$$$$$$\                              
     _\$$$$$$\| $$    $$ /      $$| $$  | $$                              
    |  \__| $$| $$$$$$$$|  $$$$$$$| $$  | $$                              
     \$$    $$ \$$     \ \$$    $$| $$  | $$                              
      \$$$$$$   \$$$$$$$  \$$$$$$$ \$$   \$$                                                                                      
     _______                                       __                     
    |       \                                     |  \                    
    | $$$$$$$\  ______    ______    ______    ____| $$  ______   _______  
    | $$__/ $$ /      \  /      \  /      \  /      $$ /      \ |       \ 
    | $$    $$|  $$$$$$\|  $$$$$$\|  $$$$$$\|  $$$$$$$|  $$$$$$\| $$$$$$$\
    | $$$$$$$\| $$   \$$| $$    $$| $$    $$| $$  | $$| $$    $$| $$  | $$
    | $$__/ $$| $$      | $$$$$$$$| $$$$$$$$| $$__| $$| $$$$$$$$| $$  | $$
    | $$    $$| $$       \$$     \ \$$     \ \$$    $$ \$$     \| $$  | $$
     \$$$$$$$  \$$        \$$$$$$$  \$$$$$$$  \$$$$$$$  \$$$$$$$ \$$   \$$

           
<br>
If you are trying to re-order Configurable Products using custom code in Magento 2.4.x or Mage-OS 1.0.x, then you might encounter this error message:<br><br>

"<strong>Selected option(s) or their combination are not currently available.</strong>"<br>

and/or:<br>

"<strong>Some item options or their combination are not currently available.</strong>"<br><br>

I have a workaround.<br>

Before the <strong>checkData()</strong> method is called, manually tell Magento that there are no errors.<br>

# Use this with extreme caution as it can mask legitimate Configurable Product errors!

This should only be used when the <strong>has_configuration_unavailable_error</strong> being set to true is causing a checkout problem with a custom configurable product.<br>

Since the error block depends on the <strong>has_configuration_unavailable_error</strong> flag being true, resetting it to false means, Magento won’t add the “<strong>unavailable configuration</strong>” errors. This workaround only disables that specific part of the code while allowing the rest of <strong>checkData()</strong> to execute as intended.<br><br>

<strong>app/code/SeanBreeden/ConfigUnavailableFix/etc/di.xml</strong>

    <?xml version="1.0"?>
    <config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
        <type name="Magento\Sales\Controller\AbstractController\Reorder">
            <plugin name="infinit_reorder_plugin" type="Infinit\Reorder\Plugin\ReorderPlugin"/>
        </type>
        <type name="Magento\Quote\Model\Quote\Item">
            <plugin name="seanbreeden_configunavailablefix_item_checkdata_plugin" type="SeanBreeden\ConfigUnavailableFix\Plugin\Quote\ItemPlugin" />
        </type>
    </config>

<strong>app/code/SeanBreeden/ConfigUnavailableFix/Plugin/Quote/ItemPlugin.php</strong>

    <?php
    namespace SeanBreeden\ConfigUnavailableFix\Plugin\Quote;

    class ItemPlugin
    {
        /**
         * @param \Magento\Quote\Model\Quote\Item $subject
         * @return void
         */
        public function beforeCheckData(\Magento\Quote\Model\Quote\Item $subject)
        {
            $subject->setData('has_configuration_unavailable_error', false);
        }
    }
    
